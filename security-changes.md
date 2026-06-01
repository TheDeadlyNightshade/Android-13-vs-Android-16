# Security & Inter-App Communication (13 → 16)

Changes to intents, components, code loading, screen capture, and platform-API access.
Most of these are hardening that closes patterns apps (and malware) historically relied on.

---

## Android 13 (API 33)

### 🟡 Non-matching intents blocked for exported components
- Intents that don't match a declared `<intent-filter>` are not delivered to exported
  components (with limited exceptions). Make internal intents **explicit**.

### 🟢 Updated non-SDK interface restrictions
- More private/internal methods/fields are blocked. Audit reflection-based access; migrate
  to public APIs. *(This list grows every release — see 14/15/16 below.)*

---

## Android 14 (API 34)

### 🟠 Restrictions on implicit & pending intents
- **Implicit intents** are delivered only to **exported** components. An implicit intent to
  a non-exported component throws `ActivityNotFoundException` — make it explicit:
  ```kotlin
  Intent("com.example.ACTION").apply { setPackage(packageName) }  // explicit
  ```
- **Mutable `PendingIntent`** without a component/package throws — specify the target or use
  `FLAG_IMMUTABLE`.

### 🟠 Runtime-registered receivers must declare export state
- `Context.registerReceiver(...)` must pass `RECEIVER_EXPORTED` or `RECEIVER_NOT_EXPORTED`
  (system broadcasts are exempt). Missing flag throws at registration.

### 🟠 Safer Dynamic Code Loading
- Dynamically-loaded code files must be **read-only before loading** (see
  [storage-and-data.md](./storage-and-data.md)). Prevents tampering between write and load.

### 🟠 `MediaProjection` requires per-session consent
- You can't cache a screen-capture intent and reuse it; each
  `getMediaProjection()` needs a fresh user-consent intent, and each `MediaProjection`
  instance creates a `VirtualDisplay` once. Register `MediaProjection.Callback` and handle
  `onStop()`. Affects screen-recording/sharing apps.

### 🟡 Background activity-start opt-ins
- See [background-execution.md](./background-execution.md): senders must opt in to grant
  background-activity-start to `PendingIntent`/bound services.

### 🟢 Zip path traversal; non-SDK list updated
- `ZipException` on `..` / leading `/` entries; more non-SDK interfaces restricted.

### ⚙️ Install-time floor (all apps)
- Devices block installing apps that **target a very old API level** (initially `minSdk`-ish
  enforcement around API 23), an anti-malware measure. Ship a current target.

---

## Android 15 (API 35)

### 🟠 Secured background activity launches (multiple)
- `PendingIntent` creators **block background activity launches by default**.
- Apps can't foreground a task via `PendingIntent` unless creator allows it or sender has
  privileges.
- Non-visible windows are excluded from launch eligibility; cross-app task injection
  ("phishing") is prevented.

### 🟡 `StrictMode.detectUnsafeIntentLaunch()`
- New StrictMode check to catch unsafe intent usage during development:
  ```kotlin
  StrictMode.setVmPolicy(StrictMode.VmPolicy.Builder()
      .detectUnsafeIntentLaunch().build())
  ```

### 🟡 OpenJDK / language-level strictness (can surface as crashes)
- Stricter `String.format()`/`Formatter` validation; `Arrays.asList().toArray()` returns
  `Object[]`; obsolete locale codes (`iw`/`ji`/`in`) no longer auto-converted; Kotlin
  `List.removeFirst()/removeLast()` can `NoSuchMethodError` on `minSdk < 35`. Audit and test.

### 🟢 Non-SDK list updated again.

---

## Android 16 (API 36)

### 🟡 Safer Intents (opt-in, becoming default later)
- New strict intent matching you can opt into:
  - Explicit intents must match the target component's intent filter.
  - Intents without an action can't match any filter.
  ```xml
  <application android:intentMatchingFlags="enforceIntentFilter">
      <receiver android:name=".MyReceiver" android:exported="true"
                android:intentMatchingFlags="allowNullAction">
          <intent-filter>
              <action android:name="com.example.MY_ACTION" />
          </intent-filter>
      </receiver>
  </application>
  ```
- Watch logcat for `Intent does not match component's intent filter` / `Access blocked`.

### 🟡 Bluetooth bond-loss / encryption intents
- New `ACTION_KEY_MISSING` (bond loss) and `ACTION_ENCRYPTION_CHANGE`; plus
  `CompanionDeviceManager.removeBond(int)` to unpair. OEM behavior varies — handle absence.

### 🟢 GPU syscall filtering (Mali)
- Deprecated/dev Mali IOCTLs blocked in production; profiling restricted to debuggable apps.
  Does **not** affect Vulkan/OpenGL. Rare for typical apps.

### 🟢 Non-SDK list updated again.

---

## All-apps security changes (apply even without raising `targetSdkVersion`)

These activate once the device runs the given OS version, regardless of your target. Don't
miss them — they can break things on a user's updated phone before you ship anything.

### Android 14 (all apps)
- 🟡 **`killBackgroundProcesses()` is self-only.** You can no longer kill other apps'
  background processes; attempts fail silently (`Invalid packageName:` in logcat).
- 🟢 **`MediaStore.OWNER_PACKAGE_NAME` may be redacted** unless the owner has an
  always-visible package name or you hold `QUERY_ALL_PACKAGES` (Play-policy gated).

### Android 15 (all apps)
- 🟠 **OTP redaction in notifications.** Untrusted `NotificationListenerService` apps can't
  read unredacted one-time-password content; only trusted (e.g. companion-device) services
  are exempt. Plan alternative flows if you relied on reading OTPs from notifications.
- 🟠 **Screen-share / screen-record protection.** Sensitive content (password fields, OTP
  notifications) is auto-hidden during capture; notification content is redacted unless
  `setPublicVersion()` is set. Use `setContentSensitivity()` to mark sensitive UI.
- 🟡 **Force-stop cancels pending intents.** Entering the stopped state cancels the app's
  pending intents and disables its widgets until next launch; re-register on
  `ACTION_BOOT_COMPLETED` and detect via `ApplicationStartInfo.wasForceStopped()`.

### Android 16 (all apps)
- 🟠 **Intent-redirection hardening (default-on).** The platform hardens against intent
  redirection exploits. Most apps are unaffected; if a legitimate nested-intent flow breaks,
  you can *opt out per intent* (use sparingly):
  ```kotlin
  val sub: Intent? = intent.getParcelableExtra("sub_intent", Intent::class.java)
  sub?.removeLaunchSecurityProtection()   // API 36+; reflection on ≤35
  sub?.let { startActivity(it) }
  ```
- 🟡 **Ordered-broadcast priority is in-process only.** `android:priority` /
  `IntentFilter.setPriority()` ordering is no longer guaranteed across processes; use real
  IPC for cross-process coordination.

## Cross-cutting theme

The arc from 13 → 16 systematically removes ways for one app (or a hidden component) to:

- receive intents it wasn't explicitly meant to (exported/implicit-intent rules),
- launch UI or services from the background invisibly (BAL + FGS restrictions),
- capture the screen or run injected code without fresh consent/integrity checks
  (`MediaProjection`, safer DCL),
- reach private platform internals (non-SDK restrictions),
- talk to LAN devices silently (local-network permission, in the networking doc).

For a normal app, the fix is always the same shape: **be explicit, be user-visible, hold the
right permission, and use public APIs.**

## Sources

- Android 14 behavior changes: https://developer.android.com/about/versions/14/behavior-changes-14
- Android 15 behavior changes: https://developer.android.com/about/versions/15/behavior-changes-15
- Android 16 behavior changes: https://developer.android.com/about/versions/16/behavior-changes-16
- Non-SDK interfaces: https://developer.android.com/guide/app-compatibility/restrictions-non-sdk-interfaces
