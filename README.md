# DANAVI Bluetooth Manager

This is a library to use DANAVI Bluetooth Manager API for Swift. By using this library, your app can easily communicate with DANAVI compatible devices using Bluetooth.

## Installation
This library is distributed through Swift Package Manager. To integrate this library to your app, do the following steps,
1. Open XCode
2. File > Swift Packages > Add Package Dependency
3. On the search bar, enter "https://github.com/dialisa-dev/DanaviBluetoothManager.git" and click Next
4. For the rules, choose Version > Up to next major, then click Next then Finish

## App's permission
Add Bluetooth permission NSBluetoothAlwaysUsageDescription to Info.plist

## Interfaces
```
public struct EmittedEvent {
    public var mac: String = "";
    public var action: String = "";
    public var deviceType: String = "";
    public var value: Any = "";
}

public struct Temperature {
    public var temperature: Double = 0.0;
    public var unit: String = "fahrenheit";
}

public struct OximeterMeasurement {
    public var bloodOxygen: Int = 0;
    public var heartRate: Int = 0;
    public var pi: Int = 0;
}

public struct SpirometerMeasurement {
    public var fev1: Double = 0.0;
    public var pef: Double = 0.0;
}
```

## API
### 1. Scan for devices
```
public func scanDevices(deviceType type: String, timeout: Double? = 10)
```
```BluetoothManager``` will scan for devices with ```deviceType``` for ```timeout``` seconds. The default value for ```timeout``` is 10.0 seconds. After ```timeout``` seconds, the ```BluetoothManager``` will send ```EmittedEvent(mac: "", action: "ACTION_SCAN_FINISHED", deviceType: "", value: "")``` to indicate that the ```BluetoothManager``` has stopped scanning for devices. 

When device is found, it will send ```EmittedEvent(mac: "49319DAB-B10E-A065-537C-F7E4AC91121A", action: "ACTION_DEVICE_DISCOVERED", deviceType: "DANAVI Oximeter", value: "")```.

### 2. Stop scanning
```
public func stopScan()
```
Calling this function will send ```EmittedEvent(mac: "", action: "ACTION_SCAN_FINISHED", deviceType: "", value: "")``` to indicate that the ```BluetoothManager``` has stopped scanning for devices.

### 3. Connect to device
```
public func connectDevice(mac: String)
```
Connect to device with MAC Address ```mac```. After a successful connection, it will send ```EmittedEvent(mac: "49319DAB-B10E-A065-537C-F7E4AC91121A", action: "ACTION_DEVICE_CONNECTED", deviceType: "DANAVI Oximeter", value: "")```

### 4. Disconnect device
```
public func disconnectDevice(mac: String)
```
Cancel connection to device with MAC Address ```mac```. After a device is disconnected, it will send ```EmittedEvent(mac: "49319DAB-B10E-A065-537C-F7E4AC91121A", action: "ACTION_DEVICE_DISCONNECTED", deviceType: "DANAVI Oximeter", value: "")```

### 5. Start DANAVI in-ear thermometer measurement
```
public func startDanaviThermometerMeasurement()
```
Call this function to start thermometer measurement. Please wait for ```EmittedEvent(mac: "49319DAB-B10E-A065-537C-F7E4AC91121A", action: "ACTION_DEVICE_READY_TO_TAKE_MEASUREMENT", deviceType: "DANAVI Oximeter", value: "")```, then prompt the user to take temperature measurement using the thermometer. 

On successful measurement, it will send ```EmittedEvent(mac: "5B5BE370-CBB2-0192-0CC0-BC8DC74D4C80", action: "ACTION_DANAVI_THERMOMETER_MEASUREMENT", deviceType: "DANAVI Ear Thermometer", value: DanaviBluetoothManager.Temperature(temperature: 98.2, unit: "fahrenheit"))```

### 6. Start DANAVI Oximeter measurement
```
public func startDanaviOximeterMeasurement(writeToFile: Bool? = false, withTimer: Bool? = false, measurementLength: Int? = 300, thinningInterval: Int? = 20)
```
Call this function to start oximeter measurement. Please wait for ```EmittedEvent(mac: "49319DAB-B10E-A065-537C-F7E4AC91121A", action: "ACTION_DEVICE_READY_TO_TAKE_MEASUREMENT", deviceType: "DANAVI Oximeter", value: "")```, then prompt the user to take oximeter measurement. 

On successful measurement, it will send ```EmittedEvent(mac: "49319DAB-B10E-A065-537C-F7E4AC91121A", action: "ACTION_OXIMETER_LIVE_DATA", deviceType: "DANAVI Oximeter", value: DanaviBluetoothManager.OximeterMeasurement(bloodOxygen: 99, heartRate: 82, pi: 5))```

#### writeToFile

Setting this parameter to ```true``` will write the oximeter measurement to a temporary file with format ```Blood Oxygen Level, Heart Rate, Pi\n```. Default value is ```false```. For example, the temporary file will look like this,
```
99,86,15
99,87,5
99,87,5
```

#### withTimer

Setting this parameter to ```true``` will set the measurement length for oximeter measurement. Default value is ```false```. After some ```measurementLength``` seconds, it will stop sending the oximeter measurement event and stop writing oximeter measurement data to a temporary file. It will send the remaining measurement length in ```EmittedEvent(mac: "DANAVI Oximeter", action: "ACTION_OXIMETER_TIMER", deviceType: "DANAVI Oximeter", value: 10)```. When the timer hits 0, it will send ```EmittedEvent(mac: "49319DAB-B10E-A065-537C-F7E4AC91121A", action: "ACTION_OXIMETER_MEASUREMENT_DONE", deviceType: "DANAVI Oximeter", value: "")``` to indicate that the oximeter measurement has finished.

#### measurementLength

Measurement length for oximeter measurement in **seconds**. Default value is 300 seconds. If ```withTimer``` is ```false```, then this parameter will not affect anything.

#### thinningInterval

```thinningInterval``` is used for thinning the received oximeter measurement data. By default, the oximeter will send **approximately** 20 data/second. Default value is 20, which means that it only takes every 20th measurement (approximately 1 data/second).

### 7. Stop DANAVI Oximeter measurement
```
public func stopDanaviOximeterMeasurement()
```
Call this function to stop sending oximeter measurement. It will send ```EmittedEvent(mac: "49319DAB-B10E-A065-537C-F7E4AC91121A", action: "ACTION_OXIMETER_MEASUREMENT_DONE", deviceType: "DANAVI Oximeter", value: "")``` to indicate that the oximeter measurement has finished.

### 8. Retrieve oximeter data from file
```
public func retrieveOximeterData() -> String
```
Call this function to retrieve oximeter data from the temporary file. In order for this to work, must set ```writeToFile``` parameter in ```startDanaviOximeterMeasurement``` function to ```true```. On failure, it will return ```-1,-1,-1```.

### 9. Start DANAVI Spirometer measurement
```
public func startDanaviSpirometerMeasurement()
```
Call this function to start spirometer measurement. Please wait for ```EmittedEvent(mac: "0759B832-91A5-FDE8-C6D3-0B5552DA5B46", action: "ACTION_DEVICE_READY_TO_TAKE_MEASUREMENT", deviceType: "DANAVI Spirometer", value: "")``` then prompt the user to take measurement. After successful measurement, you will hear 1 beep from the spirometer device and the measurement will show up on the spirometer's screen. It will take a few seconds (~5 seconds) until the measurement is sent to the app.

On successful measurement, it will send ```EmittedEvent(mac: "0759B832-91A5-FDE8-C6D3-0B5552DA5B46", action: "ACTION_SPIROMETER_MEASUREMENT", deviceType: "DANAVI Spirometer", value: DanaviBluetoothManager.SpirometerMeasurement(fev1: 2.2600000000000002, pef: 361.0))```. PEF is in L/min and FEV1 is in L.

## Workflow
### 1. Import the library
```
import DanaviBluetoothManager
```

### 2. Initialize BluetoothManager class
```
var bluetoothManager = BluetoothManager()
```

### 3. Subscribe to Event and register callbacks
Since most of the functions are asynchronous, we use ```EmittedEvent``` struct to communicate between the UIViewController and the BluetoothManager. Our API sends all type of events to ```EVENT_BLUETOOTH_DANAVI```. In the ```EmittedEvent``` struct, there is an ```action``` field to indicate what type of event that you receive.
```
override func viewDidLoad() {
    super.viewDidLoad()
    bluetoothManager.events.listenTo(eventName: EVENT_BLUETOOTH_DANAVI, action: self.onReceiveEvent)
}
    
func onReceiveEvent(information: Any?) {
    let info = information as! EmittedEvent
    print("\n\(info)")
    if (info.action == ACTION_BLUETOOTH_STATE_CHANGE) {
        onBluetoothStateChange(event: info)
    } else if (info.action == ACTION_DEVICE_DISCOVERED) {
        onDeviceDiscovered(event: info)
    } else if (info.action == ACTION_SCAN_FINISHED) {
        onScanFinished(event: info)
    } else if (info.action == ACTION_DEVICE_CONNECTED) {
        onDeviceConnected(event: info)
    } else if (info.action == ACTION_DEVICE_DISCONNECTED) {
        onDeviceDisconnected(event: info)
    } else if (info.action == ACTION_DEVICE_FAILED_TO_CONNECT) {
        onDeviceFailedToConnect(event: info)
    } else if (info.action == ACTION_DANAVI_THERMOMETER_MEASUREMENT) {
        handleReceiveTemperatureMeasurement(event: info)
    } else if (info.action == ACTION_DEVICE_READY_TO_TAKE_MEASUREMENT) {
        handleReadyTakeMeasurement()
    } else if (info.action == ACTION_OXIMETER_LIVE_DATA) {
        handleReceiveOximeterMeasurement(event: info)
    } else if (info.action == ACTION_OXIMETER_TIMER) {
        handleOximeterTimer(event: info)
    } else if (info.action == ACTION_OXIMETER_MEASUREMENT_DONE) {
        handleOximeterMeasurementDone()
    }
}
```

### 4. Handle ACTION_BLUETOOTH_STATE_CHANGE action
If the ```event.value``` is equal to ```PoweredOn```, it means that the phone's Bluetooth state is on and we can use BluetoothManager's APIs. If ```event.value``` is not equal to ```PoweredOn```, then there is a problem with the phone's Bluetooth. Possible values are ```Unknown```, ```Resetting```, ```Unsupported```, ```Unauthorized```, ```PoweredOff```, ```PoweredOn```, and ```Error```. The ```event``` structure will look like this,
```
EmittedEvent(mac: "", action: "ACTION_BLUETOOTH_STATE_CHANGE", deviceType: "", value: "PoweredOn")
```

```
func onBluetoothStateChange(event: EmittedEvent) {
    if (event.value as! String == "PoweredOn") {
        // IF BLUETOOTH STATE IS POWERED ON, THEN READY TO SCAN DEVICES
        bluetoothManager.scanDevices(deviceType: DANAVI_THERMOMETER_TYPE, timeout: 3.0)
    } else {
        // NEED TO HANDLE -- BLUETOOTH STATE IS NOT ON
    }
}
```

### 5. Scan for devices
Start scanning for nearby devices by using ```scanDevices``` function,

Sample ```EmittedEvent```
```
EmittedEvent(mac: "EDC8D22F-CEFF-F0E3-76A7-D690CD8F09C8", action: "ACTION_DEVICE_DISCOVERED", deviceType: "TS28B", value: "")

EmittedEvent(mac: "", action: "ACTION_SCAN_FINISHED", deviceType: "", value: "")
```

### 6. Connect to device
Initiate a connection to device by using ```connectDevice``` function. 

On successful connection,
```
EmittedEvent(mac: "EDC8D22F-CEFF-F0E3-76A7-D690CD8F09C8", action: "ACTION_DEVICE_CONNECTED", deviceType: "TS28B", value: "")
```

On failed connection,
```
EmittedEvent(mac: "EDC8D22F-CEFF-F0E3-76A7-D690CD8F09C8", action: "ACTION_DEVICE_FAILED_TO_CONNECT", deviceType: "TS28B", value: "")
```

### 7. Start measurement
Once the device is connected to the phone, then can start listening for measurement event by using ```startDanaviThermometerMeasurement``` function. Wait until the app receives ```EmittedEvent``` with ```ACTION_DEVICE_READY_TO_TAKE_MEASUREMENT``` ```action```, then prompt the user to use the thermometer to take measurement.

Notification event
```
EmittedEvent(mac: "EDC8D22F-CEFF-F0E3-76A7-D690CD8F09C8", action: "ACTION_DEVICE_READY_TO_TAKE_MEASUREMENT", deviceType: "TS28B", value: "")
```

### 8. Handle measurement event
After the user use the thermometer, it will generate ```EmittedEvent``` that looks like this,
```
EmittedEvent(mac: "EDC8D22F-CEFF-F0E3-76A7-D690CD8F09C8", action: "ACTION_DANAVI_THERMOMETER_MEASUREMENT", deviceType: "TS28B", value: DanaviBluetoothManager.Temperature(temperature: 98.2, unit: "fahrenheit"))
```

### 9. Cleanup
After receiving measurement, do a cleanup by ```disconnectDevice(mac)``` and ```bluetoothManager.events.removeListeners(eventNameToRemoveOrNil: EVENT_BLUETOOTH_DANAVI)```

## Sample Code
```
//
//  ViewController.swift
//  BluetoothTest2
//
//  Created by Darren Muliawan on 8/26/20.
//  Copyright © 2020 Darren Muliawan. All rights reserved.
//

import UIKit
import CoreBluetooth
import DanaviBluetoothManager

class ViewController: UIViewController {
    var bluetoothManager = BluetoothManager()
    var discoveredMac: [String] = []
    
    override func viewDidLoad() {
        super.viewDidLoad()
        bluetoothManager.events.listenTo(eventName: EVENT_BLUETOOTH_DANAVI, action: self.onReceiveEvent)
        bluetoothManager.enableBluetoothLogging()
    }
    
    // WRAPPER FUNCTION TO HANDLE ALL EVENTS SENT FROM BLUETOOTH MANAGER
    func onReceiveEvent(information: Any?) {
        let info = information as! EmittedEvent
        if (info.action == ACTION_BLUETOOTH_STATE_CHANGE) {
            print("\n\(info)")
            onBluetoothStateChange(event: info)
        } else if (info.action == ACTION_DEVICE_DISCOVERED) {
            print("\n\(info)")
            onDeviceDiscovered(event: info)
        } else if (info.action == ACTION_SCAN_FINISHED) {
            print("\n\(info)")
            onScanFinished(event: info)
        } else if (info.action == ACTION_DEVICE_CONNECTED) {
            print("\n\(info)")
            onDeviceConnected(event: info)
        } else if (info.action == ACTION_DEVICE_DISCONNECTED) {
            print("\n\(info)")
            onDeviceDisconnected(event: info)
        } else if (info.action == ACTION_DEVICE_FAILED_TO_CONNECT) {
            print("\n\(info)")
            onDeviceFailedToConnect(event: info)
        } else if (info.action == ACTION_DANAVI_THERMOMETER_MEASUREMENT) {
            print("\n\(info)")
            handleReceiveTemperatureMeasurement(event: info)
        } else if (info.action == ACTION_DEVICE_READY_TO_TAKE_MEASUREMENT) {
            print("\n\(info)")
            handleReadyTakeMeasurement()
        } else if (info.action == ACTION_OXIMETER_LIVE_DATA) {
            print("\n\(info)")
            handleReceiveOximeterMeasurement(event: info)
        } else if (info.action == ACTION_OXIMETER_TIMER) {
            //print("\n\(info)")
            handleOximeterTimer(event: info)
        } else if (info.action == ACTION_OXIMETER_MEASUREMENT_DONE) {
            print("\n\(info)")
            handleOximeterMeasurementDone()
        } else if (info.action == ACTION_DANAVI_SPIROMETER_MEASUREMENT_TAKEN) {
            print("\n\(info)")
            handleSpirometerMeasurementTaken(event: info)
        } else if (info.action == ACTION_SPIROMETER_MEASUREMENT) {
            print("\n\(info)")
            handleSpirometerMeasurement(event: info)
        }
    }
    
    // FUNCTION TO HANDLE BLUETOOTH STATE CHANGE
    func onBluetoothStateChange(event: EmittedEvent) {
        if (event.value as! String == "PoweredOn") {
            // IF BLUETOOTH STATE IS POWERED ON, THEN READY TO SCAN DEVICES
            //bluetoothManager.scanDevices(deviceType: DANAVI_EAR_THERMOMETER_TYPE, timeout: 5.0)
            bluetoothManager.scanDevices(deviceType: DANAVI_SPIROMETER_TYPE, timeout: 5.0)
        } else {
            // NEED TO HANDLE -- BLUETOOTH STATE IS NOT ON
        }
    }
    
    func onDeviceDiscovered(event: EmittedEvent) {
        // IF DEVICE IS DISCOVERED, APPEND MAC ADDRESS TO ARRAY
        self.discoveredMac.append(event.mac)
    }
    
    func onScanFinished(event: EmittedEvent) {
        // BLUETOOTH SCANNING HAS FINISHED
        if (self.discoveredMac.count == 1) {
            // ONLY 1 DEVICE FOUND, ATTEMPT TO CONNECT RIGHT AWAY
            bluetoothManager.connectDevice(mac: self.discoveredMac[0])
        } else if (self.discoveredMac.count == 0) {
            // NEED TO HANDLE -- NO DEVICE FOUND
        } else {
            // IF MORE THAN 1 DEVICE FOUND, PROMPT USER TO CHOOSE DEVICE
        }
    }
    
    func onDeviceConnected(event: EmittedEvent) {
        // IF DEVICE IS CONNECTED, THEN START MEASUREMENT
        if (event.deviceType == DANAVI_EAR_THERMOMETER_TYPE) {
            bluetoothManager.startDanaviThermometerMeasurement()
            //bluetoothManager.disconnectDevice(mac: event.mac)
        } else if (event.deviceType == DANAVI_OXIMETER_TYPE) {
            bluetoothManager.startDanaviOximeterMeasurement(writeToFile: true, withTimer: true, measurementLength: 10, thinningInterval: 20)
        } else if (event.deviceType == DANAVI_SPIROMETER_TYPE) {
            bluetoothManager.startDanaviSpirometerMeasurement()
        }
    }
    
    func onDeviceDisconnected(event: EmittedEvent) {
        // TODO: UPDATE UI
    }
    
    func onDeviceFailedToConnect(event: EmittedEvent) {
        // TODO: UPDATE UI
    }
    
    func handleReceiveTemperatureMeasurement(event: EmittedEvent) {
        // GET TEMPERATURE OBJECT
        let measurement = event.value as! Temperature
        print("Temperature: \(measurement.temperature) \(measurement.unit)")
        
        // UPDATE UI
    }
    
    func handleReceiveOximeterMeasurement(event: EmittedEvent) {
        // GET OXIMETER MEASUREMENT OBJECT
        let measurement = event.value as! OximeterMeasurement
        print("Blood Oxygen: \(measurement.bloodOxygen), Heart Rate: \(measurement.heartRate), Pi: \(measurement.pi)")
        
        // UPDATE UI
    }
    
    func handleReadyTakeMeasurement() {
        // UPDATE UI TO PROMPT USER TO USE DEVICE
    }
    
    func handleOximeterTimer(event: EmittedEvent) {
        // UPDATE UI TO SHOW THE OXIMETER MEASUREMENT REMAINING TIME
        let remainingTime = event.value
        print(remainingTime)
    }
    
    func handleOximeterMeasurementDone() {
        // UPDATE UI
        // PROMPT USER THAT THEY HAVE FINISHED THE MEASUREMENT
        let data = bluetoothManager.retrieveOximeterData().split(whereSeparator: \.isNewline)
        print("Total recorded measurement: \(data.count)\n\(data)")
    }
    
    func handleSpirometerMeasurementTaken(event: EmittedEvent) {
        // UPDATE UI
    }
    
    func handleSpirometerMeasurement(event: EmittedEvent) {
        // UPDATE UI
        let measurement = event.value as! SpirometerMeasurement
        print("FEV1: \(measurement.fev1), PEF: \(measurement.pef)")
    }
}

```
