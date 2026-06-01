# Storage & Data: Where it lives, how it's accessed (13 → 16)

Covers how app data is stored and how apps read shared media/files across the migration.

---

## Where app data lives (unchanged fundamentals)

These model basics are stable across 13–16; included for grounding:

- **App-private storage** (`Context.filesDir`, `getExternalFilesDir()`,
  `Context.dataDir`) — owned by the app, removed on uninstall, no permission needed.
- **`SharedPreferences` / Jetpack DataStore / app database (Room/SQLite)** — live in
  app-private storage.
- **Shared collections** (`MediaStore` images/video/audio/downloads) — governed by
  **scoped storage** and the media permissions below.

The big shifts 13 → 16 are in **how you reach *shared* media** and a few security-driven
constraints on writing/loading code and reading version metadata.

---

## Android 13 (API 33)

### 🟠 Granular media permissions
- `READ_EXTERNAL_STORAGE` → `READ_MEDIA_IMAGES` / `READ_MEDIA_VIDEO` / `READ_MEDIA_AUDIO`.
- Best practice: use the **Photo Picker** (no permission) instead of broad media reads.
  See [permissions-and-privacy.md](./permissions-and-privacy.md).

---

## Android 14 (API 34)

### 🟠 Selected Photos Access
- Users can grant a **subset** of photos/videos. Custom pickers must add
  `READ_MEDIA_VISUAL_USER_SELECTED`; otherwise the system runs a compatibility mode and
  your "all media" assumptions break.
- **Effect on data flows:** your app may now see only the user-selected items, and the set
  can change between sessions. Re-query rather than caching "the whole library."

### 🟠 Safer Dynamic Code Loading (storage-adjacent)
- Any file you dynamically load as code (DEX/JAR/APK) must be set **read-only before** you
  write/close it, or loading throws.
  ```kotlin
  val jar = File(dir, "plugin.jar")
  FileOutputStream(jar).use { /* write... */ }
  jar.setReadOnly()      // required before loading
  PathClassLoader(jar.path, parentLoader)
  ```
- **Effect:** if you cache downloaded code modules, change the write/seal order and verify
  integrity (signature check) before loading. Full details in
  [security-changes.md](./security-changes.md).

### 🟢 Zip path-traversal protection
- `ZipFile` / `ZipInputStream` throw `ZipException` on entries containing `..` or starting
  with `/`. Affects apps that unzip downloaded archives into storage.

---

## Android 15 (API 35)

No new *storage-model* gating beyond the background/security items, but note:

- The **edge-to-edge** change (15) and **configuration/insets** changes can affect anything
  that persisted layout geometry assumptions — not storage of data per se, but worth a
  regression pass if you cache view metrics. See
  [security-changes.md](./security-changes.md) and the official UI notes.

---

## Android 16 (API 36)

### 🟢 `MediaStore.getVersion()` is now per-app
- The value is unique per app and **must not be used for cross-app fingerprinting**. Don't
  parse its format; just treat any change as "media store changed, re-sync."

### 🟢 Photo Picker pre-selection
- App-owned photos are pre-selected when the user limits access; deselecting revokes. No
  code change.

> Networking-related data movement (TLS floor, local-network permission) is its own topic:
> see [networking-and-data-transfer.md](./networking-and-data-transfer.md).

---

## Migration map (storage & data)

| If your app… | Change | Action |
|---|---|---|
| Reads shared images/video/audio | Granular media (13) | `READ_MEDIA_*` or Photo Picker |
| Has its own media gallery UI | Selected Photos (14) | Add `READ_MEDIA_VISUAL_USER_SELECTED`; handle partial sets |
| Downloads & loads code modules | Safer DCL (14) | Seal read-only before load; verify signatures |
| Unzips downloaded archives | Zip traversal (14) | Handle `ZipException`; sanitize entry names |
| Reads `MediaStore.getVersion()` | Per-app version (16) | Don't fingerprint; re-sync on change |

## Sources

- Scoped storage / data storage: https://developer.android.com/training/data-storage
- Selected Photos Access: https://developer.android.com/about/versions/14/changes/partial-photo-video-access
- Safer dynamic code loading: https://developer.android.com/about/versions/14/behavior-changes-14#safer-dynamic-code-loading
- Android 16 behavior changes: https://developer.android.com/about/versions/16/behavior-changes-16
