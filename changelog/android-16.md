# Android 16 (API 36) — Changelog

Topic detail in [networking](../networking-and-data-transfer.md),
[permissions](../permissions-and-privacy.md), [security](../security-changes.md),
[background](../background-execution.md).

## Apps targeting Android 16

| Change | Sev | Impact | Topic |
|---|---|---|---|
| **Local Network Protection permission** | 🔴 | LAN access (mDNS/SSDP/`NsdManager`/local sockets) needs `NEARBY_WIFI_DEVICES`; remote internet unaffected | [networking](../networking-and-data-transfer.md) |
| **Edge-to-edge opt-out removed** | 🔴 | `windowOptOutEdgeToEdgeEnforcement` ignored; must support edge-to-edge | see checklist F |
| **Predictive back mandatory** | 🔴 | `onBackPressed()`/`KEYCODE_BACK` not dispatched; use `OnBackInvokedCallback` | see checklist F |
| **Large-screen: orientation/resizability ignored (≥600dp)** | 🔴 | `screenOrientation`/`resizableActivity`/aspect ratios ignored; build adaptive layouts | see checklist F |
| Granular health permissions | 🟠 | `health.*` replace `BODY_SENSORS*`; privacy-policy activity required | [permissions](../permissions-and-privacy.md) |
| Safer intents (opt-in) | 🟡 | Strict intent-filter matching via `intentMatchingFlags` | [security](../security-changes.md) |
| `scheduleAtFixedRate()` skips backlog | 🟡 | At most one missed run executes | [background](../background-execution.md) |
| Bluetooth bond-loss/encryption intents | 🟡 | New `ACTION_KEY_MISSING`/`ACTION_ENCRYPTION_CHANGE`; OEM-varying | — |
| `CompanionDeviceManager.removeBond()` | 🟢 | Public unpair API | — |
| `MediaStore.getVersion()` per-app | 🟢 | Don't fingerprint; re-sync on change | [storage](../storage-and-data.md) |
| elegantTextHeight disabled | 🟢 | Text rendering normalized across scripts | — |
| GPU syscall filtering (Mali) | 🟢 | Doesn't affect Vulkan/OpenGL | [security](../security-changes.md) |
| Photo Picker pre-selects app-owned photos | 🟢 | No code change | [permissions](../permissions-and-privacy.md) |

## All apps (on Android 16 devices)

| Change | Sev | Impact | Topic |
|---|---|---|---|
| **Intent-redirection hardening (default-on)** | 🟠 | Nested-intent exploits blocked; opt out per-intent via `removeLaunchSecurityProtection()` if needed | [security](../security-changes.md) |
| JobScheduler quota enforcement | 🟠 | Jobs subject to quotas by bucket/visibility/FGS; debug via `getStopReason()` | [background](../background-execution.md) |
| `STOP_REASON_TIMEOUT_ABANDONED` | 🟡 | Keep strong ref to `JobParameters`; reduce abandoned jobs | [background](../background-execution.md) |
| `setImportantWhileForeground()` fully deprecated | 🟢 | No-op; remove calls | — |
| Ordered broadcast priority in-process only | 🟡 | No cross-process ordering guarantee; use IPC | [security](../security-changes.md) |
| ART internal changes | 🟠 | Non-SDK/internal-ART users may break; test | [security](../security-changes.md) |
| 16 KB page-size default + compat mode | 🟠 | 4 KB apps get warning dialog; `android:pageSizeCompat` or rebuild | — |
| Accessibility announcements deprecated | 🟡 | Replace `announceForAccessibility()` with live regions/pane titles | — |
| Adaptive layouts on external displays | 🟠 | Virtual-device owners can override orientation/aspect/resizability | see checklist F |
| Companion discovery timeout → `RESULT_USER_REJECTED` | 🟢 | Update error handling | — |
| Improved Bluetooth bond-loss handling | 🟡 | Manual re-pair prompted; handle disconnection | — |
| Auto-themed app icons (QPR2+) | 🟢 | Add monochrome layer to control | — |
| 3-button nav predictive back | 🟢 | No action if migrated | — |

## Sources
- https://developer.android.com/about/versions/16/behavior-changes-16
- https://developer.android.com/about/versions/16/behavior-changes-all
