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
```

## API
### 1. Scan for devices
```
public func scanDevices(deviceType type: String, timeout: Double? = nil)
```
```BluetoothManager``` will scan for devices with ```deviceType``` for ```timeout``` seconds. The default value for ```timeout``` is 10.0 seconds. After ```timeout``` seconds, the ```BluetoothManager``` will send an ```EmittedEvent``` with action ```ACTION_SCAN_FINISHED``` to indicate that the ```BluetoothManager``` has stopped scanning for devices.

### 2. Stop scanning
```
public func stopScan()
```

### 3. Connect to device
```
public func connectDevice(mac: String)
```
Connect to device with MAC Address ```mac```

### 4. Disconnect device
```
public func disconnectDevice(mac: String)
```
Cancel connection to device with MAC Address ```mac```

### 5. Start thermometer measurement
```
public func startDanaviThermometerMeasurement()
```

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
    }
}
```

### 4. Handle ACTION_BLUETOOTH_STATE_CHANGE action
If the ```event.value``` is equal to ```PoweredOn```, it means that the phone's Bluetooth state is on and we can use BluetoothManager's APIs. If ```event.value``` is not equal to ```PoweredOn```, then there is a problem with the phone's Bluetooth. The ```event``` structure will look like this,
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
    }
    
    // WRAPPER FUNCTION TO HANDLE ALL EVENTS SENT FROM BLUETOOTH MANAGER
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
        }
    }
    
    // FUNCTION TO HANDLE BLUETOOTH STATE CHANGE
    func onBluetoothStateChange(event: EmittedEvent) {
        if (event.value as! String == "PoweredOn") {
            // IF BLUETOOTH STATE IS POWERED ON, THEN READY TO SCAN DEVICES
            bluetoothManager.scanDevices(deviceType: DANAVI_THERMOMETER_TYPE, timeout: 3.0)
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
            // NEED TO HANDLE UI -- NO DEVICE FOUND
        } else {
            // IF MORE THAN 1 DEVICE FOUND, PROMPT USER TO CHOOSE DEVICE
        }
    }
    
    func onDeviceConnected(event: EmittedEvent) {
        // IF DEVICE IS CONNECTED, THEN START MEASUREMENT
        if (event.deviceType == DANAVI_THERMOMETER_TYPE) {
            bluetoothManager.startDanaviThermometerMeasurement()
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
    
    func handleReadyTakeMeasurement() {
        // UPDATE UI TO PROMPT USER TO USE THERMOMETER
    }
}
```
