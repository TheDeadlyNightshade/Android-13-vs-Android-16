# Android 13 → Android 16: App Migration Reference

A comprehensive, organized reference for what changed between **Android 13 (API 33)**
and **Android 16 (API 36)**, and how those changes affect an existing app that was
built/targeted for Android 13 and is now updating to Android 16.

## The single most important concept

Behavior changes in Android are **cumulative** and most are **gated on
`targetSdkVersion`**. When you raise `targetSdkVersion` from **33 → 36**, your app does
not just inherit "Android 16 changes" — it inherits **every** behavior change introduced
for apps targeting **API 34 (Android 14)**, **API 35 (Android 15)**, *and* **API 36
(Android 16)**, all at once. That is the part teams most often underestimate.

So this reference is organized by **topic**, and within each topic the changes are listed
**in version order (14 → 15 → 16)** so you can see the full stack of things that activate
when you cross from 33 to 36.

## Scope & methodology (read this)

- This is a **synthesis of Google's official behavior-change documentation**, organized
  for migration. It is **not** a raw line-by-line diff of AOSP source — a full OS diff
  between two versions is millions of lines across kernel, HALs, and system services,
  ~99% of which never affects how an app behaves. The app-relevant surface is the
  **documented behavior changes** and the **public API (`android.jar`) changes**, which
  is what this covers.
- Every document links its **official sources** at the bottom. Because details evolve
  (especially for the most recent release) and because I am summarizing, **verify
  anything load-bearing against the linked official pages** before you ship.

## Documents

| Doc | Covers |
|-----|--------|
| [00-overview.md](./00-overview.md) | API/version map, how `targetSdkVersion` gating works, the two change "buckets", how to test changes with the compatibility framework, severity legend |
| [permissions-and-privacy.md](./permissions-and-privacy.md) | Notifications, granular media, photo picker / selected media, nearby Wi-Fi, body-sensor → granular health permissions, full-screen intents, local network |
| [background-execution.md](./background-execution.md) | Foreground service types & timeouts, `BOOT_COMPLETED` restrictions, background activity/service starts, JobScheduler, exact alarms, audio focus |
| [storage-and-data.md](./storage-and-data.md) | Where app data lives, scoped storage & media access, selected-photos access, `MediaStore` version, dynamic code loading, backup |
| [networking-and-data-transfer.md](./networking-and-data-transfer.md) | How apps send/receive data: TLS floor, **local network permission (16)**, network-state permission for jobs, cleartext defaults |
| [security-changes.md](./security-changes.md) | Implicit/pending intents, receiver export flags, safer intents, `MediaProjection` consent, non-SDK restrictions, install-time minimum target SDK |
| [migration-checklists.md](./migration-checklists.md) | Practical "if your app does X on 13, here's what breaks at 16 and how to fix it" checklists, manifest audit, test plan |

## How to use this

1. Start with **[00-overview.md](./00-overview.md)** to understand the gating model.
2. Read **[migration-checklists.md](./migration-checklists.md)** for a fast triage of what
   likely affects your app.
3. Drill into the topic docs for the changes the checklist flagged.

## Official sources (top level)

- Android 14 behavior changes (apps targeting 14): https://developer.android.com/about/versions/14/behavior-changes-14
- Android 15 behavior changes (apps targeting 15): https://developer.android.com/about/versions/15/behavior-changes-15
- Android 16 behavior changes (apps targeting 16): https://developer.android.com/about/versions/16/behavior-changes-16
- Android 13 behavior changes (apps targeting 13): https://developer.android.com/about/versions/13/behavior-changes-13
- "All apps" variants (changes that apply regardless of target): replace `behavior-changes-NN` with `behavior-changes-all` in the URLs above.

*Compiled as a documentation project. Verify against official docs before relying on any specific detail.*
