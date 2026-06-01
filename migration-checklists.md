# Migration Checklists: Android 13 → Android 16

Practical triage for an app currently targeting **API 33** that is moving to **API 36**.
Work top-to-bottom; each row links to the topic doc with detail.

---

## A. Build & manifest baseline

- [ ] Set `compileSdk = 36` and `targetSdk = 36`; raise `minSdk` to a still-supported level.
- [ ] Recompile and resolve new lint/`NewApi` warnings (especially Kotlin
      `removeFirst()/removeLast()` if `minSdk < 35`). → [security](./security-changes.md)
- [ ] Update AndroidX/Jetpack, build tools, and third-party SDKs (many platform changes are
      actually *SDK* changes you inherit).
- [ ] Confirm you don't ship below the install-time target floor. → [security](./security-changes.md)

## B. Foreground services & background work (highest-risk area)

- [ ] Every foreground service declares a correct `foregroundServiceType` + permission (14).
- [ ] `dataSync`/`mediaProcessing` FGS implement `onTimeout()` and respect the **6h/24h**
      cap (15).
- [ ] No `BOOT_COMPLETED` receiver starts a restricted FGS type (15).
- [ ] Background FGS via `SYSTEM_ALERT_WINDOW` only with a **visible** overlay (15).
- [ ] `onStartJob()/onStopJob()` return in seconds; heavy work moved to WorkManager (14).
- [ ] `JobInfo` network constraints declare `ACCESS_NETWORK_STATE` (14).
- [ ] Re-validate periodic/fixed-rate scheduling cadence (16).
- [ ] Audio-focus requests happen only in foreground/FGS (15).
→ [background-execution.md](./background-execution.md)

## C. Permissions & privacy

- [ ] `POST_NOTIFICATIONS` requested + denial handled (13).
- [ ] Media reads use `READ_MEDIA_*` or Photo Picker; custom pickers add
      `READ_MEDIA_VISUAL_USER_SELECTED` (13/14).
- [ ] Full-screen-intent notifications gated on `canUseFullScreenIntent()` (14).
- [ ] Heart-rate/SpO₂/temperature use `health.*` permissions + a privacy-policy activity (16).
- [ ] Wi-Fi APIs use `NEARBY_WIFI_DEVICES` with `neverForLocation` (13).
→ [permissions-and-privacy.md](./permissions-and-privacy.md)

## D. Networking & data transfer

- [ ] All endpoints/SDKs support **TLS ≥ 1.2** (15).
- [ ] Cleartext only via narrowly-scoped Network Security Config.
- [ ] **Local-network access** (mDNS/SSDP/`NsdManager`/sockets to LAN IPs) requests
      `NEARBY_WIFI_DEVICES` and degrades gracefully on denial (16).
- [ ] Remote-internet traffic verified unaffected by the local-network rule (16).
→ [networking-and-data-transfer.md](./networking-and-data-transfer.md)

## E. Intents, components & security

- [ ] Internal intents are explicit; exported flags correct on all components (13/14).
- [ ] Runtime receivers pass `RECEIVER_EXPORTED`/`RECEIVER_NOT_EXPORTED` (14).
- [ ] Mutable `PendingIntent`s specify component/package or are `FLAG_IMMUTABLE` (14).
- [ ] Dynamic code loading seals files read-only + verifies signatures (14).
- [ ] `MediaProjection` re-consents per session; `Callback.onStop()` handled (14).
- [ ] Background activity launches reworked to notification-tap flows (14/15).
- [ ] Reflection/private-API usage audited against the non-SDK list (every release).
→ [security-changes.md](./security-changes.md)

## F. UI / layout (will visibly break if skipped)

- [ ] Edge-to-edge insets handled everywhere; `setStatusBarColor()`/`setNavigationBarColor()`
      no longer relied on (enforced 15, **opt-out removed 16**).
- [ ] Back navigation migrated to `OnBackInvokedCallback`/`BackEventCompat`; `onBackPressed()`
      no longer relied on (16).
- [ ] Large-screen (≥600dp) layouts adapt — `screenOrientation`/`resizableActivity`/aspect
      ratios are **ignored** at 16; don't lock orientation. → see Android 16 source.
- [ ] Don't use `Configuration.screenWidthDp/HeightDp` for layout math (now includes system
      bars, 15) — use `WindowInsets`/`WindowMetricsCalculator`.

---

## "If your app does X" quick lookup

| Your app… | Most likely to break on | Go to |
|---|---|---|
| Runs a persistent background sync | FGS types (14), 6h cap (15), BOOT_COMPLETED (15) | [background](./background-execution.md) |
| Records/streams the screen | `MediaProjection` per-session consent (14) | [security](./security-changes.md) |
| Discovers/controls devices on Wi-Fi | Local-network permission (16) | [networking](./networking-and-data-transfer.md) |
| Picks photos/media | Granular media (13), Selected Photos (14) | [permissions](./permissions-and-privacy.md) |
| Uses heart-rate / health sensors | `health.*` permissions (16) | [permissions](./permissions-and-privacy.md) |
| Talks to a legacy backend | TLS 1.2 floor (15), cleartext default | [networking](./networking-and-data-transfer.md) |
| Uses overlays / launches UI in background | BAL restrictions (14/15) | [background](./background-execution.md) |
| Loads plugins / downloaded code | Safer DCL (14) | [security](./security-changes.md) |
| Targets phones only, single orientation | Large-screen adaptive layouts (16) | Android 16 source |

---

## G. Test plan

1. **Two device matrix:** test on an Android 16 device/emulator (targeting bucket) *and*
   confirm older-OS behavior with your new build (all-apps bucket).
2. **Toggle changes individually** with the compat framework before flipping `targetSdk`,
   e.g. `adb shell am compat enable FGS_INTRODUCE_TIME_LIMITS <pkg>`. →
   [00-overview.md](./00-overview.md)
3. **Watch logcat** for: `ForegroundServiceStartNotAllowedException`,
   `ActivityNotFoundException`, `SecurityException`, StrictMode unsafe-intent violations,
   FGS timeout `RemoteServiceException`, and local-network socket errors (`EPERM`).
4. **Exercise denial paths:** deny each runtime permission and verify graceful degradation.
5. **Diagnose ANRs/crashes** post-mortem via `ApplicationExitInfo.getTraceInputStream()`.

## Sources

See each linked topic doc; top-level index in the [README](./README.md).
