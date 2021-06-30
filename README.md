## [Documentation](https://zhortech.github.io/ztcorekit-android-sdk)

## Getting Started

The first step is to include general ztcorekit and device specific modules into your project.

### Gradle

If you use Gradle to build your project — as a Gradle project implementation dependency:
```groovy
implementation "fr.zhortech.android:ztcorekit:$zhortechSdkVersion-prod"
```
for each of your build type declare set of buildConfigField that with values that you receive from Zhortech:
```groovy
buildConfigField("String", "API_KEY", "...")
buildConfigField("String", "API_SECRET", "...")
buildConfigField("String", "APP_ID", "...")
buildConfigField("String", "APP_TYPE", "...")
```
### Maven
TODO

### Permissions
Android since API 23 (6.0 / Marshmallow) requires location permissions declared in the manifest for an app to run a BLE scan. ztcorekit already provides all the necessary bluetooth permissions for you in AndroidManifest.

Runtime permissions required for running a BLE scan:

| from API | to API (inclusive) | Acceptable runtime permissions |
|:---:|:---:| --- |
| 18 | 22 | (No runtime permissions needed) |
| 23 | 28 | One of below:<br>- `android.permission.ACCESS_COARSE_LOCATION`<br>- `android.permission.ACCESS_FINE_LOCATION` |
| 29 | current | - `android.permission.ACCESS_FINE_LOCATION` |

## Usage

### Application setup
Inside you application class `onCreate()` add following code:
```kotlin
val product = //instance of firmware specific product
val appInfo = ZTAppInformation(
    BuildConfig.API_KEY, 
    BuildConfig.API_SECRET, 
    BuildConfig.APP_ID, 
    BuildConfig.APP_TYPE
    )
ZTCore(
    this,
    appInfo,
    product
)
```
You can override default settings in ZTSettings by setting corresponding fields.

### User identification.
Application has to identify user using method -
`ZTApi.linkUser(userId: String, payload: Map<String, Any>)`

It will create user object in portal for new user or associate user with current session to allows viewing activities in future.

## BLE connection
### Connect
For first connection you should obtain `ZTDevice` by providing scanned QR code
```kotlin
ZTBleManager.getBleDeviceWithCode("Dawd")
    .observeOn(AndroidSchedulers.mainThread())
    .subscribe(
        { device ->
            //code parsed successfully
        },
        {
            //error
        }
    )
```
or you can check if there are any already connected `ZTDevice` by `ZTBleManager.getConnectedDevices()`.

Afterwards you connect to device:
```kotlin
ZTBleManager.connectToDevice(device)
    .observeOn(AndroidSchedulers.mainThread())
    .subscribe(
        { connectedDevice ->
            //device connected successfully
        },
        {
            //error
        }
    )
```
After successful connection SDK will remember device. So you can connect by:
```kotlin
ZTBleManager.reconnectLastDevice()
```
### Disconnect
Disconnection can be explicit or unexpected.
If you need to disconnect from peripheral, use this method:
```kotlin
device.disconnect()
```
Unexpected can be due for different reasons: device out of range, low battery, device reset etc.

### Auto reconnect
After application initialize connection, SDK will handle unexpected disconnects and will try to reconnect to device if `ZTSettings.autoReconnect` is enabled.

## DFU

After first connection application it is mandatory to check device firmware for compatibility and new firmware version:
```kotlin
 val product = ZTBleManager.getProduct()
 product.checkFirmwareUpdate()
        .observeOn(AndroidSchedulers.mainThread())
        .subscribe(
            { 
                val dfuData = it.get()
                //if dfuData is null you have latest firmware
            },
            {
                //error
            }
        )
```
To start DFU call:
```kotlin
 val dfuInitiator = ZTDfuInitiator(ZTCore.context, dfuData)
dfuInitiator.registerProgressListener(dfuProgressListener)

dfuInitiator.start()
    .observeOn(AndroidSchedulers.mainThread())
    .subscribe(
        {
            //dfu complete  
        },
        {
            //error
            if (it is ZTLowBatteryException) {
                //low battery
            }
        }
    )
```
