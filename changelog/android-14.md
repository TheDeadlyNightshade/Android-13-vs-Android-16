# Android 14 (API 34) — Changelog

Topic detail in [background](../background-execution.md),
[security](../security-changes.md), [permissions](../permissions-and-privacy.md),
[FGS matrix](../reference/foreground-service-types.md).

## Apps targeting Android 14

| Change | Sev | Impact | Topic |
|---|---|---|---|
| **Foreground service types mandatory** | 🔴 | Every FGS needs a `foregroundServiceType` + permission, else `MissingForegroundServiceTypeException` | [FGS matrix](../reference/foreground-service-types.md) |
| Implicit intents → exported only | 🟠 | Implicit intent to non-exported component throws `ActivityNotFoundException` | [security](../security-changes.md) |
| Mutable `PendingIntent` needs component/package | 🟠 | Else exception; or use `FLAG_IMMUTABLE` | [security](../security-changes.md) |
| Runtime receivers need export flag | 🟠 | `RECEIVER_EXPORTED`/`RECEIVER_NOT_EXPORTED` required | [security](../security-changes.md) |
| Safer dynamic code loading | 🟠 | DCL files must be read-only before load | [security](../security-changes.md) |
| `MediaProjection` per-session consent | 🟠 | Fresh consent per session; one `VirtualDisplay` per instance | [security](../security-changes.md) |
| Background activity-start opt-ins | 🟠 | `PendingIntent.send()` / bound services must opt in | [background](../background-execution.md) |
| Selected Photos Access | 🟠 | Custom pickers add `READ_MEDIA_VISUAL_USER_SELECTED` | [storage](../storage-and-data.md) |
| JobScheduler callback timeouts | 🟠 | `onStartJob/onStopJob` ANR if slow; move to WorkManager | [background](../background-execution.md) |
| JobScheduler network needs `ACCESS_NETWORK_STATE` | 🟡 | `SecurityException` otherwise | [networking](../networking-and-data-transfer.md) |
| Full-screen-intent restricted | 🟡 | Only calling/alarm apps auto-granted | [permissions](../permissions-and-privacy.md) |
| `BLUETOOTH_CONNECT` enforced on `getProfileConnectionState()` | 🟡 | Declare + check permission | [permissions](../permissions-and-privacy.md) |
| OpenJDK 17 strictness (regex/UUID/ProGuard) | 🟡 | `IllegalArgumentException` in new cases; test | [security](../security-changes.md) |
| Zip path traversal | 🟢 | `ZipException` on `..` / leading `/` | [storage](../storage-and-data.md) |
| `TileService.startActivityAndCollapse(Intent)` | 🟢 | Use `PendingIntent` overload | [background](../background-execution.md) |
| Non-SDK restrictions updated | 🟠 | More private APIs blocked | [security](../security-changes.md) |

## All apps (on Android 14 devices)

| Change | Sev | Impact | Topic |
|---|---|---|---|
| **Min installable target API = 23** | 🟠 | `INSTALL_FAILED_DEPRECATED_SDK_VERSION` below 23 | [security](../security-changes.md) |
| Exact alarms denied by default | 🟠 | `SCHEDULE_EXACT_ALARM` not pre-granted; handle denial | [background](../background-execution.md) |
| `killBackgroundProcesses()` self-only | 🟡 | Can't kill other apps' processes | [security](../security-changes.md) |
| Non-dismissible notifications dismissible | 🟡 | `setOngoing(true)` user-dismissible (with exceptions) | — |
| Context broadcasts queued while cached | 🟡 | May merge; delivered when uncached | — |
| New restricted-bucket reason (job ANRs) | 🟡 | Repeated job ANRs → restricted bucket | [background](../background-execution.md) |
| BLE ATT MTU = 517 on first request | 🟡 | Cap GATT writes to `min(MTU,517)-5` | — |
| `mlock()` capped at 64 KB | 🟢 | Refactor large locked regions | — |
| `MediaStore.OWNER_PACKAGE_NAME` redacted | 🟢 | Unless visible pkg or `QUERY_ALL_PACKAGES` | [security](../security-changes.md) |
| Data safety shown in more places | 🟢 | Keep Play data-safety accurate | — |
| Non-linear font scaling to 200% | 🟢 | Use `sp`; test large fonts | — |

## Sources
- https://developer.android.com/about/versions/14/behavior-changes-14
- https://developer.android.com/about/versions/14/behavior-changes-all
- https://developer.android.com/about/versions/14/changes/fgs-types-required
