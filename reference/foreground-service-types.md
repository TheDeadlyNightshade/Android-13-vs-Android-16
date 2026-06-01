# Reference: Foreground Service Type → Permission Matrix

Since **Android 14 (API 34)**, every foreground service must declare a
`android:foregroundServiceType`, and each type requires a specific
`FOREGROUND_SERVICE_*` permission **plus** (often) one or more runtime prerequisites.
This is the complete mapping, with the Android 15 additions and restrictions noted.

## Always required (base)

```xml
<uses-permission android:name="android.permission.FOREGROUND_SERVICE" />
```
- `FOREGROUND_SERVICE` is a **normal** permission (granted at install, non-revocable),
  required for *any* foreground service since API 28.
- The per-type `FOREGROUND_SERVICE_*` permissions below are **also normal** permissions —
  granted by default, can't be revoked — but they **must be declared** or the service
  won't start.

## The matrix

| Type (`foregroundServiceType`) | Required `FOREGROUND_SERVICE_*` permission | Runtime prerequisites / conditions | Typical use cases |
|---|---|---|---|
| `camera` | `FOREGROUND_SERVICE_CAMERA` | `CAMERA` granted (while-in-use) | Camera use continuing in background (video chat + multitask) |
| `connectedDevice` | `FOREGROUND_SERVICE_CONNECTED_DEVICE` | At least one of `CHANGE_NETWORK_STATE`, `CHANGE_WIFI_STATE`, `CHANGE_WIFI_MULTICAST_STATE`, `NFC`, `TRANSMIT_IR`; **or** granted `BLUETOOTH_CONNECT`/`BLUETOOTH_ADVERTISE`/`BLUETOOTH_SCAN`/`UWB_RANGING`; **or** `UsbManager.requestPermission()` | Bluetooth/NFC/IR/USB/network device interactions |
| `dataSync` | `FOREGROUND_SERVICE_DATA_SYNC` | None | Upload/download, backup-restore, import/export, local file processing. **⚠ 6h/24h cap (15); Google steers you to alternatives** |
| `health` | `FOREGROUND_SERVICE_HEALTH` | Declare `HIGH_SAMPLING_RATE_SENSORS`; **or** granted `BODY_SENSORS` (API ≤35), `READ_HEART_RATE`, `READ_SKIN_TEMPERATURE`, `READ_OXYGEN_SATURATION`, or `ACTIVITY_RECOGNITION` | Fitness/exercise tracking |
| `location` | `FOREGROUND_SERVICE_LOCATION` | Location services on; `ACCESS_COARSE_LOCATION` or `ACCESS_FINE_LOCATION` granted; `ACCESS_BACKGROUND_LOCATION` for background | Navigation, location sharing |
| `mediaPlayback` | `FOREGROUND_SERVICE_MEDIA_PLAYBACK` | None | Background audio/video playback; Android TV DVR |
| `mediaProjection` | `FOREGROUND_SERVICE_MEDIA_PROJECTION` | Call `MediaProjectionManager.createScreenCaptureIntent()` → get user consent → `getMediaProjection()` **per session** | Screen capture/cast to another display/device |
| `microphone` | `FOREGROUND_SERVICE_MICROPHONE` | `RECORD_AUDIO` granted (while-in-use) | Background mic capture (recorders, comms) |
| `phoneCall` | `FOREGROUND_SERVICE_PHONE_CALL` | `MANAGE_OWN_CALLS` **or** default dialer (`ROLE_DIALER`) | Ongoing calls via `ConnectionService` |
| `remoteMessaging` | `FOREGROUND_SERVICE_REMOTE_MESSAGING` | None | Transfer messages between devices / device continuity |
| `mediaProcessing` *(added API 35)* | `FOREGROUND_SERVICE_MEDIA_PROCESSING` | None | Transcoding/processing. **⚠ 6h/24h cap (15)** |
| `shortService` | **None** (only base `FOREGROUND_SERVICE`) | ~3-minute timeout; **can't** start other FGS; not sticky; must implement `Service.onTimeout()` | Critical, brief, non-interruptible finish-up work |
| `specialUse` | `FOREGROUND_SERVICE_SPECIAL_USE` | Declare subtype via `<property android:name="android.app.PROPERTY_SPECIAL_USE_FGS_SUBTYPE" .../>`; justify in Play Console | Valid FGS use not covered by any other type |
| `systemExempted` | `FOREGROUND_SERVICE_SYSTEM_EXEMPTED` | App must qualify: demo mode, Device Owner, Profile Owner, `ROLE_EMERGENCY`, Device Admin, exact-alarm holder (`SCHEDULE_EXACT_ALARM`/`USE_EXACT_ALARM`), or VPN | Reserved system integrations |

## Manifest + start pattern

```xml
<uses-permission android:name="android.permission.FOREGROUND_SERVICE" />
<uses-permission android:name="android.permission.FOREGROUND_SERVICE_LOCATION" />
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />

<service
    android:name=".TrackingService"
    android:foregroundServiceType="location"
    android:exported="false" />
```

```kotlin
// Runtime permissions (e.g. ACCESS_FINE_LOCATION) must already be GRANTED
// before this call, or startForeground throws SecurityException.
ServiceCompat.startForeground(
    this, NOTIF_ID, notification,
    ServiceInfo.FOREGROUND_SERVICE_TYPE_LOCATION
)
```

## Failure modes (know the exceptions)

| Symptom | Cause |
|---|---|
| `MissingForegroundServiceTypeException` | Targeting 14+ and no `foregroundServiceType` declared |
| `SecurityException` at `startForeground()` | Required runtime prerequisite (e.g. `CAMERA`, `RECORD_AUDIO`) not granted, or `FOREGROUND_SERVICE_*` permission not declared |
| `ForegroundServiceStartNotAllowedException` | Started from background without an allowed reason; or `BOOT_COMPLETED`/`SYSTEM_ALERT_WINDOW` restrictions (15) |
| `RemoteServiceException: ... did not stop within its timeout` | `dataSync`/`mediaProcessing` exceeded the 6h/24h cap and didn't `stopSelf()` after `onTimeout()` |

## Cross-references

- 6-hour timeout, `BOOT_COMPLETED` restrictions, SAW restriction, background-start rules →
  [../background-execution.md](../background-execution.md)
- "While-in-use" types (`camera`, `microphone`, `location`, health sensors) **can't be
  started while the app is in the background** unless the app has a specific exemption.
- Health permissions are further granularized in Android 16 →
  [../permissions-and-privacy.md](../permissions-and-privacy.md)

## Sources

- Foreground service types are required (Android 14): https://developer.android.com/about/versions/14/changes/fgs-types-required
- Foreground service types overview: https://developer.android.com/develop/background-work/services/fgs/service-types
- FGS timeouts (Android 15): https://developer.android.com/about/versions/15/behavior-changes-15#fgs-timeout
