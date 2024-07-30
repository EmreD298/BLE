# Bluetooth İşlemleri - Flutter Kodları

Bu döküman, FlutterFlow'da Bluetooth ile ilgili temel işlemleri gerçekleştiren kod parçalarını ve açıklamalarını içermektedir.

## `checkBluetoothState`

```dart
import 'package:flutter_blue_plus/flutter_blue_plus.dart';

Future<bool> checkBluetoothState() async {
  if (await FlutterBluePlus.isSupported == false) {
    print("Bluetooth not supported by this device");
    return false;
  } else
    return true;
}
```
**Açıklama:** Bu fonksiyon, cihazın Bluetooth desteğinin olup olmadığını kontrol eder. Eğer cihaz Bluetooth'u desteklemiyorsa false döner ve bir uyarı mesajı yazdırır, aksi takdirde true döner.

## `connectDevice`

```dart
import 'package:flutter_blue_plus/flutter_blue_plus.dart';

Future<bool> connectDevice(BTDeviceStruct deviceInfo) async {
  final device = BluetoothDevice.fromId(deviceInfo.id);
  try {
    await device.connect();
  } catch (e) {
    print(e);
  }

  var hasWriteCharacteristic = false;
  final services = await device.discoverServices();
  for (BluetoothService service in services) {
    for (BluetoothCharacteristic characteristic in service.characteristics) {
      final isWrite = characteristic.properties.write;
      if (isWrite) {
        debugPrint(
            'Found write characteristic ${characteristic.uuid}, ${characteristic.properties}');
        hasWriteCharacteristic = true;
      }
    }
  }
  return hasWriteCharacteristic;
}
```
**Açıklama:** Bu fonksiyon, belirtilen Bluetooth cihazına bağlanmayı dener ve cihazın yazma karakteristiğine sahip olup olmadığını kontrol eder. Bağlanma başarılı olursa ve yazma karakteristiği bulunursa true döner, aksi takdirde false döner.

## `disconnectDevice`

```dart
import 'package:flutter_blue_plus/flutter_blue_plus.dart';

Future disconnectDevice(BTDeviceStruct deviceInfo) async {
  final device = BluetoothDevice.fromId(deviceInfo.id);
  try {
    await device.disconnect();
  } catch (e) {
    print(e);
  }
}
```
**Açıklama:** Bu fonksiyon, belirtilen Bluetooth cihazının bağlantısını keser. Bağlantıyı kesme işlemi sırasında bir hata oluşursa bu hata mesajını konsola yazdırır.

## `findDevices`

```dart
import 'package:flutter_blue_plus/flutter_blue_plus.dart';

Future<List<BTDeviceStruct>> findDevices() async {
  List<BTDeviceStruct> devices = [];
  FlutterBluePlus.scanResults.listen((results) {
    List<ScanResult> scannedDevices = [];
    for (ScanResult r in results) {
      if (r.device.name.isNotEmpty) {
        scannedDevices.add(r);
      }
    }
    scannedDevices.sort((a, b) => b.rssi.compareTo(a.rssi));
    devices.clear();
    scannedDevices.forEach((deviceResult) {
      devices.add(BTDeviceStruct(
        name: deviceResult.device.name,
        id: deviceResult.device.id.toString(),
        rssi: deviceResult.rssi,
      ));
});
  });

  final isScanning = FlutterBluePlus.isScanningNow;
  if (!isScanning) {
    await FlutterBluePlus.startScan(
      timeout: const Duration(seconds: 5),
    );
  }

  return devices;
}
```
**Açıklama:** Bu fonksiyon, yakındaki Bluetooth cihazlarını tarar ve taranan cihazları bir liste halinde döner. Cihazlar sinyal gücüne (RSSI) göre sıralanır.

## `getConnectedDevices`

```dart
import 'package:flutter_blue_plus/flutter_blue_plus.dart';

Future<List<BTDeviceStruct>> getConnectedDevices() async {
  final connectedDevices = await FlutterBluePlus.connectedDevices;
  List<BTDeviceStruct> devices = [];
  for (int i = 0; i < connectedDevices.length; i++) {
    final deviceResult = connectedDevices[i];
    final deviceState = await deviceResult.state.first;
    if (deviceState == BluetoothDeviceState.connected) {
      final rssi = await deviceResult.readRssi();
      devices.add(BTDeviceStruct(
        name: deviceResult.name,
        id: deviceResult.id.toString(),
        rssi: rssi,
      ));
    }
  }

  return devices;
}
```
**Açıklama:** Bu fonksiyon, şu anda bağlı olan Bluetooth cihazlarını döner. Her cihazın adı, kimliği ve RSSI değeri listelenir.

## `getRssi`

```dart
import 'package:flutter_blue_plus/flutter_blue_plus.dart';

Future<int> getRssi(BTDeviceStruct deviceInfo) async {
  final device = BluetoothDevice.fromId(deviceInfo.id);
  return await device.readRssi();
}
```
**Açıklama:** Bu fonksiyon, belirtilen Bluetooth cihazının RSSI (sinyal gücü) değerini döner.

## `isBluetoothEnabled`

```dart
import 'package:flutter_blue_plus/flutter_blue_plus.dart';

Future<bool> isBluetoothEnabled() async {
  final state = await FlutterBluePlus.adapterState.first;
  return state == BluetoothAdapterState.on;
}
```
**Açıklama:** Bu fonksiyon, cihazın Bluetooth'unun açık olup olmadığını kontrol eder. Bluetooth açıksa true, değilse false döner.

## `sendData`

```dart
import 'package:flutter_blue_plus/flutter_blue_plus.dart';

Future sendData(BTDeviceStruct deviceInfo, String data) async {
  final device = BluetoothDevice.fromId(deviceInfo.id);
  final services = await device.discoverServices();
  for (BluetoothService service in services) {
    for (BluetoothCharacteristic characteristic in service.characteristics) {
      final isWrite = characteristic.properties.write;
      if (isWrite) {
        await characteristic.write(data.codeUnits);
      }
    }
  }
}
```
**Açıklama:** Bu fonksiyon, belirtilen Bluetooth cihazına veri gönderir. Cihazın yazma karakteristiğine sahip olduğundan emin olur ve ardından veriyi yazar.

## `turnOffBluetooth`

```dart
import 'package:flutter_blue_plus/flutter_blue_plus.dart';

Future<bool> turnOffBluetooth() async {
  if (await checkBluetoothState()) {
    await FlutterBluePlus.turnOff();
    return true;
  } else
    return false;
}
```
**Açıklama:** Bu fonksiyon, Bluetooth'u kapatmayı dener. Öncelikle cihazın Bluetooth desteği olup olmadığını kontrol eder, eğer destekliyorsa Bluetooth'u kapatır ve true döner, aksi takdirde false döner.

## `turnOnBluetooth`

```dart
import 'package:flutter_blue_plus/flutter_blue_plus.dart';

Future<bool> turnOnBluetooth() async {
  if (checkBluetoothState()) {
    await FlutterBluePlus.turnOn();
    return true;
  } else
    return false;
}
```
**Açıklama:** Bu fonksiyon, Bluetooth'u açmayı dener. Öncelikle cihazın Bluetooth desteği olup olmadığını kontrol eder, eğer destekliyorsa Bluetooth'u açar ve true döner, aksi takdirde false döner.

## `receiveData`

```dart
import 'package:flutter_blue_plus/flutter_blue_plus.dart';

Future<String?> receiveData(BTDeviceStruct deviceInfo) async {
  final device = BluetoothDevice.fromId(deviceInfo.id);
  final services = await device.discoverServices();
  for (BluetoothService service in services) {
    for (BluetoothCharacteristic characteristic in service.characteristics) {
      final isRead = characteristic.properties.read;
      final isNotify = characteristic.properties.notify;
      if (isRead && isNotify) {
        final value = await characteristic.read();
        return String.fromCharCodes(value);
      }
    }
  }
  return null;
}
```
**Açıklama:** Bu fonksiyon, belirtilen Bluetooth cihazından veri okumak için kullanılır. Cihazın servislerini bulur ve servislerin karakteristiklerini kontrol eder. Eğer karakteristik "read" ve "notify" özelliklerine sahipse veriyi okur, aksi takdirde "null" döner.
