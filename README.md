# node-simconnect

[![npm version](https://badge.fury.io/js/node-simconnect.svg)](https://badge.fury.io/js/node-simconnect)
[![Strict TypeScript Checked](https://badgen.net/badge/TS/Strict 'Strict TypeScript Checked')](https://www.typescriptlang.org)

A non-official SimConnect client library for Node.JS, written in TypeScript. Integrates directly with the SimConnect protocol and runs on Windows, Linux and Mac.

## Credits

This project is a rewrite of the Java client library
[jsimconnect](https://github.com/mharj/jsimconnect), originally written by
[lc0277](https://www.fsdeveloper.com/forum/members/lc0277.1581).
Details about the protocol can be found on [lc0277's old website](http://web.archive.org/web/20090620063532/http://lc0277.nerim.net/jsimconnect/doc/flightsim/simconnect/package-summary.html#package_description).

## Getting started

> :bulb: Tip: check out the [msfs-simconnect-api-wrapper](https://www.npmjs.com/package/msfs-simconnect-api-wrapper) which provides a more user-friendly wrapper around some of the `node-simconnect` APIs.

1. `npm install node-simconnect`
2. Check out the [/samples](https://github.com/EvenAR/node-simconnect/tree/master/samples) folder for example scripts.
3. Refer to the [official SimConnect documentation](https://docs.flightsimulator.com/html/Programming_Tools/SimConnect/SimConnect_API_Reference.htm) for comprehensive details on SimConnect APIs and usage.

There are also [auto generated API-docs](https://evenar.github.io/node-simconnect/).

### General introduction

You always start by calling [`open(...)`](https://evenar.github.io/node-simconnect/functions/open.html) which will attempt to open a connection with the SimConnect server (your flight simulator). If this succeeds you will get access to:

-   [`recvOpen`](https://evenar.github.io/node-simconnect/classes/RecvOpen.html): contains simulator information
-   [`handle`](https://evenar.github.io/node-simconnect/classes/SimConnectConnection.html): will be used for all SimConnect interactions

Example:

```js
import { open, Protocol } from 'node-simconnect';

const EVENT_ID_PAUSE = 1;

open('My SimConnect client', Protocol.FSX_SP2)
    .then(function ({ recvOpen, handle }) {
        console.log('Connected to', recvOpen.applicationName);
        handle.subscribeToSystemEvent(EVENT_ID_PAUSE, 'Pause');
        handle.on('event', function (recvEvent) {
            switch (recvEvent.clientEventId) {
                case EVENT_ID_PAUSE:
                    console.log(recvEvent.data === 1 ? 'Sim paused' : 'Sim unpaused');
                    break;
            }
        });
        handle.on('exception', function (recvException) {
            console.log(recvException);
        });
        handle.on('quit', function () {
            console.log('Simulator quit');
        });
        handle.on('close', function () {
            console.log('Connection closed unexpectedly (simulator CTD?)');
        });
    })
    .catch(function (error) {
        console.log('Connection failed:', error);
    });
```

### Running over network?

1. Open `SimConnect.xml`.

    - FSX: `X:\Users\<USER>\AppData\Roaming\Microsoft\FSX`
    - MSFS: `X:\Users\<USER>\AppData\Local\Packages\Microsoft.FlightSimulator_**********\LocalCache`.

1. Set property `<Address>0.0.0.0</Address>`. Example of a working SimConnect.xml file:

    ```xml
    <?xml version="1.0" encoding="Windows-1252"?>
    <SimBase.Document Type="SimConnect" version="1,0">
        <Filename>SimConnect.xml</Filename>
        <SimConnect.Comm>
            <Protocol>IPv4</Protocol>
            <Scope>local</Scope>
            <Port>5111</Port>
            <MaxClients>64</MaxClients>
            <MaxRecvSize>41088</MaxRecvSize>
            <Address>0.0.0.0</Address>
        </SimConnect.Comm>
    </SimBase.Document>
    ```

1. Provide the IP address of the simulator PC and the port number when calling `open`:

    ```js
    const options = { remote: { host: 'localhost', port: 5111 } };

    open('My SimConnect client', Protocol.FSX_SP2, options).then(/* ... */).catch(/* try again? */);
    ```

If no connection options are specified, node-simconnect will auto-discover connection details in the following order:

1. Look for a [`SimConnect.cfg`](https://docs.flightsimulator.com/html/Programming_Tools/SimConnect/SimConnect_CFG_Definition.htm) in the folder where Node.js is located. If the script is running in Electron, this will be the folder where the Electron executable is installed.
1. Look for a `SimConnect.cfg` in the user's home directory (`%USERPROFILE%`, eg. `C:\Users\<username>`)
1. Look for a named pipe in the Windows registry, automatically set by the simulator
1. Look for a port number in the Windows registry, automatically set by the simulator. node-simconnect will then connect to `localhost:<port>`.

## Functionality

Most of the APIs described in the [official SimConnect documentation](https://docs.flightsimulator.com/html/Programming_Tools/SimConnect/SimConnect_API_Reference.htm) are implemented in `node-simconnect`. For information on how each feature works, please refer to the official documentation.

Several new features have been added to the SimConnect API after the new Microsoft Flight Simulator was released, and more features are likely to come. Most of these will only be implemented on request. If you are missing any features in `node-simconnect` feel free to [open a new issue](https://github.com/EvenAR/node-simconnect/issues) or create a pull request.

Prepar3D support and Prepar3D-only-features will not be prioritized.

⚠️ = deprecated and will not work, according to the official documentation.

### Events and data

| Method                              | Tested FSX | Tested P3D | Tested MSFS |
| ----------------------------------- | ---------- | ---------- | ----------- |
| `addClientEventToNotificationGroup` |            |            |             |
| `addToClientDataDefinition`         |            |            |             |
| `addToDataDefinition`               | ✅         |            |             |
| `clearClientDataDefinition`         |            |            |             |
| `clearDataDefinition`               | ✅         |            |             |
| `clearInputGroup`                   |            |            |             |
| `clearNotificationGroup`            |            |            |             |
| `createClientData`                  |            |            |             |
| `enumerateInputEvents`              | n/a        | n/a        | ✅          |
| `enumerateInputEventParams`         | n/a        | n/a        | ✅          |
| `getInputEvent`                     | n/a        | n/a        |             |
| `mapClientDataNameToID`             |            |            |             |
| `mapClientEventToSimEvent`          |            |            |             |
| `mapInputEventToClientEvent`        |            |            |             |
| `mapInputEventToClientEventEx1`     |            |            |             |
| `removeClientEvent`                 |            |            |             |
| `removeInputEvent`                  |            |            |             |
| `requestClientData`                 |            |            |             |
| `requestDataOnSimObject`            | ✅         |            |             |
| `requestDataOnSimObjectType`        | ✅         |            |             |
| `requestNotificationGroup`          |            |            |             |
| `requestReservedKey`                |            |            |             |
| `subscribeToSystemEvent`            | ✅         |            |             |
| `setClientData`                     |            |            |             |
| `setDataOnSimObject`                | ✅         |            |             |
| `setInputEvent`                     | n/a        | n/a        |             |
| `setInputGroupPriority`             |            |            |             |
| `setInputGroupState`                |            |            |             |
| `setSystemEventState`               |            |            |             |
| `setSystemState`                    |            |            |             |
| `subscribeInputEvent`               | n/a        | n/a        |             |
| `unsubscribeFromSystemEvent`        | ✅         |            |             |

### AI Objects

| Method                       | Tested FSX | Tested P3D | Tested MSFS |
| ---------------------------- | ---------- | ---------- | ----------- |
| `aICreateParkedATCAircraft`  |            |            |             |
| `aICreateEnrouteATCAircraft` |            |            |             |
| `aICreateNonATCAircraft`     |            |            |             |
| `aICreateSimulatedObject`    |            |            |             |
| `aIReleaseControl`           |            |            |             |
| `aIRemoveObject`             |            |            |             |
| `aISetAircraftFlightPlan`    |            |            |             |

### Flights

| Method           | Tested FSX | Tested P3D | Tested MSFS |
| ---------------- | ---------- | ---------- | ----------- |
| `flightLoad`     |            |            |             |
| `flightPlanLoad` |            |            |             |
| `flightSave`     |            |            |             |

### Facility

| Method                       | Tested FSX | Tested P3D | Tested MSFS |
| ---------------------------- | ---------- | ---------- | ----------- |
| `addToFacilityDefinition`    | n/a        | n/a        | ✅          |
| `requestFacilitiesList`      |            |            |             |
| `requestFacilitiesListEx1`   | n/a        | n/a        | ✅          |
| `requestFacilityData`        | n/a        | n/a        | ✅          |
| `requestFacilityDataEx1`     | n/a        | n/a        | ✅          |
| `requestJetwayData`          | n/a        | n/a        | ✅          |
| `subscribeToFacilities`      | ✅         |            |             |
| `subscribeToFacilitiesEx1`   | n/a        | n/a        | ✅          |
| `unSubscribeToFacilities`    | ✅         |            |             |
| `unSubscribeToFacilitiesEx1` | n/a        | n/a        | ✅          |

### Menu

| Method              | Tested FSX | Tested P3D | Tested MSFS |
| ------------------- | ---------- | ---------- | ----------- |
| `menu`              | ✅         |            | ⚠️          |
| `menuAddItem`       |            |            | ⚠️          |
| `menuAddSubItem`    |            |            | ⚠️          |
| `menuDeleteItem`    |            |            | ⚠️          |
| `menuDeleteSubItem` |            |            | ⚠️          |

### Weather

| Method                                      | Tested FSX | Tested P3D | Tested MSFS |
| ------------------------------------------- | ---------- | ---------- | ----------- |
| `weatherRequestObservationAtNearestStation` | ✅         |            | ⚠️          |
| `weatherRequestCloudState`                  | ✅         |            | ⚠️          |
| `weatherRequestInterpolatedObservation`     |            |            | ⚠️          |
| `weatherRequestObservationAtStation`        |            |            | ⚠️          |
| `weatherCreateStation`                      |            |            | ⚠️          |
| `weatherSetObservation`                     |            |            | ⚠️          |
| `weatherSetModeServer`                      |            |            | ⚠️          |
| `weatherSetModeTheme`                       |            |            | ⚠️          |
| `weatherSetModeGlobal`                      |            |            | ⚠️          |
| `weatherSetModeCustom`                      |            |            | ⚠️          |
| `weatherSetDynamicUpdateRate`               |            |            | ⚠️          |
| `weatherCreateThermal`                      |            |            | ⚠️          |
| `weatherRemoveThermal`                      |            |            | ⚠️          |

### Misc

| Method                         | Tested FSX | Tested P3D | Tested MSFS |
| ------------------------------ | ---------- | ---------- | ----------- |
| `cameraSetRelative6DOF`        |            |            |             |
| `completeCustomMissionAction`  |            |            |             |
| `executeMissionAction`         |            |            |             |
| `requestSystemState`           | ✅         |            |             |
| `setNotificationGroupPriority` |            |            |             |
| `text`                         | ✅         |            | ⚠️          |
| `executeAction`                | n/a        | n/a        | ️           |
