# 00 — Overview: API map & how change-gating works

## Version ↔ API level map

| Marketing name | API level | `targetSdkVersion` value | Notes |
|---|---|---|---|
| Android 13 | 33 | `33` | Starting point for this guide |
| Android 14 | 34 | `34` | Crossed on the way to 16 |
| Android 15 | 35 | `35` | Crossed on the way to 16 |
| Android 16 | 36 | `36` | Target |

Going 33 → 36 means you opt into the **34, 35, and 36** behavior-change sets together.

## The two buckets of behavior changes

Google splits every release's changes into two groups. You must account for **both**:

1. **Changes for apps *targeting* the new version** (`behavior-changes-NN`)
   - Activate **only when you raise `targetSdkVersion`** to that level.
   - These are the ones this migration guide focuses on, because bumping 33 → 36 turns
     on three releases' worth of them at once.

2. **Changes for *all apps*** (`behavior-changes-all`)
   - Apply **regardless of `targetSdkVersion`** once the device runs that OS version.
   - You can be affected by these **even without recompiling**, simply because a user's
     phone updated. Don't skip them.

> Practical implication: some breakage appears the moment a user updates their OS (all-apps
> bucket); other breakage appears only after *you* ship a build with a higher target
> (targeting bucket). Test both paths.

## Why "cumulative" matters

Behavior-change gates are evaluated against your **current** `targetSdkVersion`, and the
gates from older releases never go away. An app that jumps 33 → 36 has effectively never
been subjected to the 34 or 35 gates before, so on first launch of the new build it can
trip:

- Android 14 gates (e.g., **mandatory foreground-service types**, implicit-intent limits)
- Android 15 gates (e.g., **edge-to-edge enforcement**, FGS timeouts)
- Android 16 gates (e.g., **local-network permission**, forced edge-to-edge, large-screen
  orientation rules)

…all on the same release. Plan the migration as "33→34, then 34→35, then 35→36," even if
you do it in one version bump, so nothing is missed.

## How to test a specific change before flipping your target

Android ships an **app-compatibility framework** that lets you toggle individual gated
changes on/off by their **change ID**, so you can validate one behavior at a time without
changing your whole target. General form:

```bash
# Turn a specific change ON for your app (simulate the new behavior)
adb shell am compat enable  <CHANGE_ID> <your.package.name>

# Turn it back OFF
adb shell am compat disable <CHANGE_ID> <your.package.name>

# Reset to defaults
adb shell am compat reset   <your.package.name>
```

Examples of change IDs referenced in later docs:

- `FGS_INTRODUCE_TIME_LIMITS` — Android 15 foreground-service time limits
- `FGS_BOOT_COMPLETED_RESTRICTIONS` — Android 15 `BOOT_COMPLETED` FGS restrictions
- `FGS_SAW_RESTRICTIONS` — Android 15 `SYSTEM_ALERT_WINDOW` FGS restriction
- `RESTRICT_LOCAL_NETWORK` — Android 16 local-network permission enforcement
- `STPE_SKIP_MULTIPLE_MISSED_PERIODIC_TASKS` — Android 16 fixed-rate scheduling change
- `DISALLOW_INVALID_GROUP_REFERENCE`, `ENABLE_STRICT_VALIDATION` — Android 14 OpenJDK 17 toggles

> Not every change has a toggle; some are unconditional once you target the version. Where
> a change ID exists, the relevant topic doc lists it.

## Severity legend used in this repo

| Badge | Meaning |
|---|---|
| 🔴 Critical | Will likely break a core flow or crash; must address before shipping |
| 🟠 High | Commonly hit; address during migration |
| 🟡 Medium | Conditional; address if you use the affected API |
| 🟢 Low | Edge case / cosmetic / informational |

## Sources

- Android 16 behavior changes: https://developer.android.com/about/versions/16/behavior-changes-16
- Android 15 behavior changes: https://developer.android.com/about/versions/15/behavior-changes-15
- Android 14 behavior changes: https://developer.android.com/about/versions/14/behavior-changes-14
- Android 13 behavior changes: https://developer.android.com/about/versions/13/behavior-changes-13
- App compatibility framework: https://developer.android.com/guide/app-compatibility/test-debug
