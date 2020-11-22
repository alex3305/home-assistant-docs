# Retrieving Xiaomi Home tokens

For several Xiaomi devices it is necessary that you set an access token. This is a 32 character long hash that is generated every time the device first connects to wifi.

## Applicable devices

This is applicable to all Xiaomi Home devices that connect over wifi. In my personal situation I retrieved the tokens of my:

* Xiaomi Roborock S5 Max
* Xiaomi Fan 1C
* Xiaomi Fan 2S

## Retrieving your token(s)

Currently it is impossible to retrieve this token officially, but there is a workaround available.

1. Install the latest Mi Home app from the Google Play Store
2. Login with your Xiaomi account
3. Set up your Xiaomi device
4. Uninstall Mi Home from your Android device
5. Install [Mi Home 5.4.49](https://www.apkmirror.com/apk/xiaomi-inc/mihome/mihome-5-4-49-release/)
6. Again login with your Xiaomi account
7. Set up ADB access to your Android device
8. Use `adb shell` to get into the shell of your Android device
9. Retrieve the tokens through `cat /sdcard/SmartHome/logs/plug_DeviceManager/*.log`

!!! note
    Replace the Xiaomi Mi Home app afterwards. Older versions may not work correctly with new(er) devices.
