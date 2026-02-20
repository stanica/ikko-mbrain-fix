# ikko-mbrain-fix

Magisk module that patches MediaTek mBrain telemetry log spam in `system_server` on iKKO MindOne (MT6789/Skyroam) devices.

## Problem

MediaTek's `MBrainLocalService` (in `/system_ext/framework/mediatek-services.jar`) calls `ApplicationHelper.isPackageManagerInitialized()` every ~6 seconds on the `android.ui` thread. On iKKO/Skyroam devices, the mBrain service context is never properly initialized, so `PackageManager` stays null and the following error fires continuously:

```
E ApplicationHelper: Fail to get PackageManager
```

This produces 7,000+ error log entries per logcat buffer, drowning out useful logs.

## Fix

Removes the `Slog.e()` call from `isPackageManagerInitialized()` in `com.mediatek.mbrainlocalservice.helper.ApplicationHelper`. The method's actual logic (return true/false based on whether `PackageManager` was initialized) is unchanged — it just stops logging about it.

### Before

```java
public static boolean isPackageManagerInitialized() {
    if (mPackageManager == null) {
        if (retry <= 3) {
            Slog.e("ApplicationHelper", "Fail to get PackageManager");
        }
        return false;
    }
    return true;
}
```

### After

```java
public static boolean isPackageManagerInitialized() {
    if (mPackageManager == null) {
        return false;
    }
    return true;
}
```

## Install

1. Download `mbrain-log-fix-magisk.zip` from [Releases](../../releases)
2. Open Magisk app, go to Modules, and flash the ZIP
3. Reboot

## Uninstall

Remove the module from the Magisk app, or boot into safe mode (hold volume-down during boot) to disable all Magisk modules.

## Tested On

- iKKO MindOne, firmware v2.112.5.92(1204)
- Android 15 (SDK 35), MediaTek MT6789
- Magisk 30.6

## Details

- Patched file: `/system_ext/framework/mediatek-services.jar`
- Patched class: `com.mediatek.mbrainlocalservice.helper.ApplicationHelper`
- The mBrain service is already non-functional on this device (`ro.vendor.mbrain.mode` defaults to `none`). This module just silences the error log from a broken initialization check that runs on a timer regardless of the mode setting.
