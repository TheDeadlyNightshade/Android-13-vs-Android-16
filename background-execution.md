# Background Execution: Services, Jobs, Alarms (13 → 16)

This is the single biggest source of migration pain from 13 → 16. Each release tightened
how, when, and for how long apps may run work in the background. Listed in version order.

---

## Android 13 (API 33)

### 🟡 Restricted App Standby Bucket loses boot broadcasts
- Apps the user (or system) places in the **restricted** background state do **not** receive
  `BOOT_COMPLETED` or `LOCKED_BOOT_COMPLETED`.
- **Effect:** don't rely on boot broadcasts for critical init; have a foreground entry path.

### 🟢 Foreground-service notification visibility
- If the user denied `POST_NOTIFICATIONS`, the FGS notification is hidden from the drawer
  (still shown in Task Manager). The service still runs; the user just may not *see* it.

---

## Android 14 (API 34) — the big one

### 🔴 Foreground service **types are mandatory**
- Every `<service>` used as a foreground service must declare at least one
  `android:foregroundServiceType`, and you must hold the matching permission.
- Types include: `camera`, `connectedDevice`, `dataSync`, `health`, `location`,
  `mediaPlayback`, `mediaProjection`, `microphone`, `phoneCall`, `remoteMessaging`,
  `shortService`, `specialUse`, `systemExempted`.
- If your use case doesn't fit a type, **migrate the work to WorkManager** or a
  user-initiated data-transfer job.

```xml
<service
    android:name=".SyncService"
    android:foregroundServiceType="dataSync"
    android:exported="false" />
```

### 🟠 Background activity-start restrictions tightened
- **`PendingIntent.send()`**: the *sender* must opt in to grant its background-activity-start
  privileges:
  ```kotlin
  val opts = ActivityOptions.makeBasic()
      .setPendingIntentBackgroundActivityStartMode(
          ActivityOptions.MODE_BACKGROUND_ACTIVITY_START_ALLOWED)
  pendingIntent.send(context, 0, null, null, null, null, opts.toBundle())
  ```
- **Service binding**: a visible app binding a background app's service must pass
  `Context.BIND_ALLOW_ACTIVITY_STARTS` to let that service start activities.

### 🟠 JobScheduler: stricter callbacks + network permission
- `onStartJob()` / `onStopJob()` must return **within a few seconds** or you get an ANR
  ("No response to onStartJob"). Move blocking work off the callback thread (prefer
  WorkManager).
- Using `setRequiredNetworkType()` / `setRequiredNetwork()` now requires
  `ACCESS_NETWORK_STATE` or scheduling throws `SecurityException`.

### 🟡 Quick Settings tile launches
- `TileService.startActivityAndCollapse(Intent)` throws; use the `PendingIntent` overload.

---

## Android 15 (API 35) — service lifetime limits

### 🔴 `dataSync` foreground service: 6-hour/24h cap
- A `dataSync` FGS may run at most **6 hours total per 24-hour window**. Then the system
  calls `Service.onTimeout(int, int)` and you must `stopSelf()` within seconds, or:
  `RemoteServiceException: A foreground service of type dataSync did not stop within its timeout`.
- The window **resets when the user foregrounds** the app.
- Test:
  ```bash
  adb shell am compat enable FGS_INTRODUCE_TIME_LIMITS <pkg>
  adb shell device_config put activity_manager data_sync_fgs_timeout_duration 3600000
  ```

### 🟠 New `mediaProcessing` FGS type (same 6h cap)
- For transcoding/processing; declare it and implement `onTimeout()`. WorkManager is the
  alternative for long jobs.

### 🔴 `BOOT_COMPLETED` cannot launch certain FGS types
- A `BOOT_COMPLETED` receiver may **not** start these FGS types:
  `dataSync`, `camera`, `mediaPlayback`, `phoneCall`, `mediaProjection`, `microphone`.
- Attempting it throws `ForegroundServiceStartNotAllowedException`. Defer until the user
  interacts with the app.
- Test: `adb shell am compat enable FGS_BOOT_COMPLETED_RESTRICTIONS <pkg>`

### 🟠 `SYSTEM_ALERT_WINDOW` no longer a free background-FGS pass
- To start an FGS from the background using overlay privileges you must have an **already
  visible** `TYPE_APPLICATION_OVERLAY` window — not just the permission. Else
  `ForegroundServiceStartNotAllowedException`.
- Test: `adb shell am compat enable FGS_SAW_RESTRICTIONS <pkg>`

### 🟡 Audio focus requires foreground
- `requestAudioFocus()` fails unless app is top or running an FGS.

### 🟡 More background-activity-launch hardening
- `PendingIntent` creators block background activity launches **by default**; non-visible
  windows are excluded from launch eligibility; cross-app task injection is blocked.
  (See [security-changes.md](./security-changes.md).)

---

## Android 16 (API 36)

### 🟡 `scheduleAtFixedRate()` skips backlog
- After the app returns to a valid lifecycle state, a backlogged fixed-rate task runs **at
  most once** (not once per missed period).
- Test: `STPE_SKIP_MULTIPLE_MISSED_PERIODIC_TASKS`.

### 🟢 JobScheduler quota refinements
- Android 16 continues tightening job quotas/regular-job behavior; revalidate that periodic
  and constrained jobs still fire on the cadence you expect, and prefer WorkManager's
  policies over hand-rolled scheduling.

---

## The throughline: what to do instead

| Old pattern (works on 13) | Why it breaks by 16 | Modern replacement |
|---|---|---|
| Untyped foreground service | FGS type mandatory (14) | Declare correct `foregroundServiceType` + permission |
| Long-running `dataSync` FGS forever | 6h/24h cap (15) | Split work; implement `onTimeout()`; use WorkManager |
| Start FGS from `BOOT_COMPLETED` | Type restrictions (15) | Defer to first user interaction; use WorkManager/`setExpedited` |
| Overlay perm to start hidden background work | SAW FGS restriction (15) | Require a visible overlay, or rethink the flow |
| Heavy work in `onStartJob()` | ANR (14) | WorkManager `CoroutineWorker`/`Worker` |
| Background activity launches via PendingIntent | Blocked by default (14/15) | Post a notification; let the user tap to foreground |

> Rule of thumb for 13 → 16: **if work needs to run when the app isn't visible, use
> WorkManager**, and reserve foreground services for genuinely user-visible ongoing tasks
> with the correct type.

## Sources

- Android 14 FGS types: https://developer.android.com/about/versions/14/changes/fgs-types-required
- Android 15 FGS timeouts: https://developer.android.com/about/versions/15/behavior-changes-15#fgs-timeout
- Android 15 behavior changes: https://developer.android.com/about/versions/15/behavior-changes-15
- Android 16 behavior changes: https://developer.android.com/about/versions/16/behavior-changes-16
- WorkManager: https://developer.android.com/topic/libraries/architecture/workmanager
