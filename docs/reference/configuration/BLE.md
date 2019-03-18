<h2 id="configuration-ble">BLE</h2>

This page describes build-time configurable parameters for BLE. A resource constrained device can limit its functionality to decrease the memory and flash use of the BLE module. 

To understand what each feature provides, please refer to the API docs and the Bluetooth specification.

#### Build time configuration of the BLE feature

To minimize the size of the BLE stack, BLE defines a set of build options.

The configuration is contained in the `mbed_lib.json` configuration file located in `mbed-os/features/FEATURE_BLE/`. By default, all the features are enabled.

Turning off individual features removes the code and any memory allocations required by that feature. Some features depend on one another - please see the comments in the configuration file for details. These dependencies are enforced during compile time.

Trying to use a disabled feature results in a compile time error or an error reporting the feature as unimplemented at run time.

#### Changing the configuration

An excerpt from the configuration file:

```
{
    "name": "ble",
    "config": {
        "ble-role-observer": {
            "help": "Include observer BLE role support, allows listening for and processing advertising packets.",
            "value": true,
            "macro_name": "BLE_ROLE_OBSERVER"
        },
    ...
```

By changing `"value": true,` to `false`, you can disable each feature.

##### Configurable features

These are feature that you can disable:

```
Configuration parameters
------------------------
```
