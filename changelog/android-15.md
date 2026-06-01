# Android 15 (API 35) — Changelog

Topic detail in [background](../background-execution.md),
[security](../security-changes.md), [networking](../networking-and-data-transfer.md).

## Apps targeting Android 15

| Change | Sev | Impact | Topic |
|---|---|---|---|
| **Edge-to-edge enforced** | 🔴 | Content draws behind bars/cutout; `setStatusBarColor`/`setNavigationBarColor` deprecated; apply insets | see migration checklist F |
| `dataSync` FGS 6h/24h cap | 🔴 | `onTimeout()` → `stopSelf()` or `RemoteServiceException` | [background](../background-execution.md) |
| `BOOT_COMPLETED` can't start restricted FGS types | 🔴 | `ForegroundServiceStartNotAllowedException` for dataSync/camera/media*/phoneCall/mic | [background](../background-execution.md) |
| New `mediaProcessing` FGS type (6h cap) | 🟠 | Declare type; implement `onTimeout()` | [FGS matrix](../reference/foreground-service-types.md) |
| `SYSTEM_ALERT_WINDOW` background FGS restricted | 🟠 | Needs a *visible* overlay window | [background](../background-execution.md) |
| Secured background activity launches (×5) | 🟠 | PendingIntent BAL blocked by default; non-visible windows excluded; anti-phishing | [security](../security-changes.md) |
| TLS 1.0/1.1 disallowed | 🟠 | Must use TLS ≥ 1.2 | [networking](../networking-and-data-transfer.md) |
| Audio focus requires foreground/FGS | 🟡 | Background `requestAudioFocus()` fails | [background](../background-execution.md) |
| DND global-state changes | 🟡 | Use `AutomaticZenRule` not `setInterruptionFilter()` | — |
| `StrictMode.detectUnsafeIntentLaunch()` | 🟡 | New dev-time intent safety check | [security](../security-changes.md) |
| OpenJDK: String.format/Arrays/locale/Sequenced | 🟡 | Stricter validation; Kotlin `removeFirst/Last` crash on `minSdk<35` | [security](../security-changes.md) |
| Display cutout = ALWAYS | 🟡 | Set `layoutInDisplayCutoutMode` explicitly | — |
| `Configuration` includes system bars | 🟡 | Don't use for layout math; use `WindowInsets` | — |
| elegantTextHeight / TextView width / EditText line height | 🟢 | Text metrics shift for complex scripts | — |
| Non-SDK restrictions updated | 🟠 | More private APIs blocked | [security](../security-changes.md) |

## All apps (on Android 15 devices)

| Change | Sev | Impact | Topic |
|---|---|---|---|
| **Min installable target API = 24** | 🟠 | `INSTALL_FAILED_DEPRECATED_SDK_VERSION` below 24 | [security](../security-changes.md) |
| **16 KB page-size support** | 🟠 | Rebuild NDK/native libs for 16 KB devices | — |
| Background network access restricted | 🟠 | Network calls outside valid lifecycle throw; make lifecycle-aware | [networking](../networking-and-data-transfer.md) |
| Force-stop cancels pending intents | 🟡 | Widgets disabled; re-register on `ACTION_BOOT_COMPLETED` | [security](../security-changes.md) |
| OTP redaction in notifications | 🟠 | Untrusted listeners can't read OTPs | [security](../security-changes.md) |
| Screen-share/record protection | 🟠 | Sensitive UI auto-hidden; `setContentSensitivity()` | [security](../security-changes.md) |
| Private space | 🟠 | Don't assume non-main profile == work; launcher/medical apps adapt | — |
| Predictive back animations on | 🟡 | Finish migration to predictive back | — |
| Direct/offload AudioTrack invalidation | 🟡 | Handle invalidated tracks | — |
| PNG emoji font removed | 🟢 | Replace `NotoColorEmojiLegacy.ttf` references | — |

## Sources
- https://developer.android.com/about/versions/15/behavior-changes-15
- https://developer.android.com/about/versions/15/behavior-changes-all
