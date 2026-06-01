# Changelog (by version)

The same changes as the topic docs, but listed **strictly per Android version** — useful
when you want "what does crossing into API N turn on?" Each page splits changes into the
two buckets explained in [../00-overview.md](../00-overview.md):

- **Targeting `<version>`** — activates when you raise `targetSdkVersion`.
- **All apps** — activates on the device OS version regardless of your target.

| Version | API | Page |
|---|---|---|
| Android 13 | 33 | [android-13.md](./android-13.md) — *baseline* |
| Android 14 | 34 | [android-14.md](./android-14.md) |
| Android 15 | 35 | [android-15.md](./android-15.md) |
| Android 16 | 36 | [android-16.md](./android-16.md) |

Severity legend: 🔴 Critical · 🟠 High · 🟡 Medium · 🟢 Low (see
[../00-overview.md](../00-overview.md)).

> Migrating 33 → 36? You inherit **android-14 + android-15 + android-16** "targeting"
> changes all at once, plus every "all apps" change on the user's device. Read all three.
