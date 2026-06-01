# Android 13 (API 33) — Changelog

This is the **baseline** for this guide. Listed so you can confirm your app already adopted
these before moving on. Topic detail in [permissions](../permissions-and-privacy.md),
[background](../background-execution.md), [security](../security-changes.md).

## Apps targeting Android 13

| Change | Sev | Impact | Topic |
|---|---|---|---|
| `POST_NOTIFICATIONS` runtime permission | 🟠 | Must request to show notifications; FGS notice hidden if denied | [permissions](../permissions-and-privacy.md) |
| Granular media permissions | 🟠 | `READ_MEDIA_IMAGES/VIDEO/AUDIO` replace `READ_EXTERNAL_STORAGE` | [storage](../storage-and-data.md) |
| `NEARBY_WIFI_DEVICES` permission | 🟡 | Wi-Fi APIs no longer need fine location (`neverForLocation`) | [permissions](../permissions-and-privacy.md) |
| `BODY_SENSORS_BACKGROUND` | 🟡 | Hard-restricted; needed for background sensor reads | [permissions](../permissions-and-privacy.md) |
| Media controls from `PlaybackState` | 🟡 | Buttons derived from `PlaybackState` actions, not `MediaStyle` | — |
| Restricted bucket → no boot broadcasts | 🟡 | No `BOOT_COMPLETED`/`LOCKED_BOOT_COMPLETED` in restricted state | [background](../background-execution.md) |
| WebView auto dark theme | 🟢 | `setForceDark()` no-op; uses app `isLightTheme` | — |
| `BluetoothAdapter.enable()/disable()` deprecated | 🟢 | Return `false` (except owners/system) | — |
| `AD_ID` permission | 🟢 | Required for real advertising ID, else zeroed | [networking](../networking-and-data-transfer.md) |
| Non-matching intents blocked (exported) | 🟡 | Intents must match filters for exported components | [security](../security-changes.md) |
| Non-SDK interface restrictions updated | 🟢 | More private APIs blocked | [security](../security-changes.md) |

## All apps (on Android 13 devices)

Notable: themed app icons (opt-in monochrome layer), per-app language preferences, and
clipboard/preview privacy. These are largely additive or cosmetic; verify icon + language
handling if relevant.

## Sources
- https://developer.android.com/about/versions/13/behavior-changes-13
- https://developer.android.com/about/versions/13/behavior-changes-all
