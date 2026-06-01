# Permissions & Privacy (13 → 16)

How permission requirements and privacy controls changed, in version order. If you are
jumping 33 → 36, **all** of these apply.

---

## Baseline established at Android 13 (API 33)

These are already in effect at your starting point, but listed because apps migrating *to*
13 originally had to adopt them — confirm you actually did.

### 🟠 Notification runtime permission (`POST_NOTIFICATIONS`)
- Posting notifications now requires the runtime `POST_NOTIFICATIONS` permission.
- **Effect on apps:** if the user denies it, your notifications (including the notice for a
  **foreground service**) won't show in the drawer — though the FGS still appears in Task
  Manager. You must request it at an appropriate moment and handle denial.

### 🟠 Granular media permissions (replaces `READ_EXTERNAL_STORAGE`)
- `READ_EXTERNAL_STORAGE` is replaced by per-type permissions:
  - `READ_MEDIA_IMAGES`, `READ_MEDIA_VIDEO`, `READ_MEDIA_AUDIO`
- Legacy apps that had `READ_EXTERNAL_STORAGE` are auto-granted the `READ_MEDIA_*` set on
  upgrade, but a fresh target-33 build must request them explicitly.
- **Recommendation:** prefer the **Photo Picker** (no permission needed) for images/video.

### 🟡 `NEARBY_WIFI_DEVICES` permission
- Wi-Fi APIs that used to require `ACCESS_FINE_LOCATION` can now use `NEARBY_WIFI_DEVICES`.
- Add `android:usesPermissionFlags="neverForLocation"` and assert you don't derive location.

```xml
<uses-permission
    android:name="android.permission.NEARBY_WIFI_DEVICES"
    android:usesPermissionFlags="neverForLocation" />
```

### 🟡 `BODY_SENSORS_BACKGROUND` for background sensor access
- Background access to heart rate / temperature / SpO₂ requires `BODY_SENSORS_BACKGROUND`,
  a **hard-restricted** permission (must be allowlisted by the installer). *(See Android 16,
  where this whole area is replaced by granular health permissions.)*

### 🟢 `AD_ID` permission for advertising ID
- Using Google Play services advertising ID requires
  `com.google.android.gms.permission.AD_ID`. Without it the ID is zeroed out.

---

## Android 14 (API 34)

### 🟠 Selected Photos Access (partial media grant)
- Users can grant access to **specific** photos/videos instead of all media.
- If you keep a custom gallery picker (instead of the system Photo Picker), declare the new
  `READ_MEDIA_VISUAL_USER_SELECTED` permission; otherwise the system runs your app in a
  compatibility mode.

```xml
<!-- Full access (old) -->
<uses-permission android:name="android.permission.READ_MEDIA_IMAGES" />
<!-- Plus: user-selected subset (new) -->
<uses-permission android:name="android.permission.READ_MEDIA_VISUAL_USER_SELECTED" />
```

### 🟡 Full-screen-intent permission restricted (`USE_FULL_SCREEN_INTENT`)
- Auto-grant on install is now limited to **calling/alarm** apps. Other apps must check
  `NotificationManager.canUseFullScreenIntent()` and, if needed, send the user to
  `Settings.ACTION_MANAGE_APP_USE_FULL_SCREEN_INTENT`.

### 🟡 `BLUETOOTH_CONNECT` now enforced on `getProfileConnectionState()`
- Declare and runtime-check `BLUETOOTH_CONNECT` before calling it, or you'll be denied.

---

## Android 15 (API 35)

Android 15's privacy-adjacent changes are mostly in **background execution** and
**security** (see those docs). The notable permission-flavored item:

### 🟡 Audio focus requires foreground
- An app can only **gain** audio focus when it's the top app or running a foreground
  service. Background focus requests return `AUDIOFOCUS_REQUEST_FAILED`.

---

## Android 16 (API 36)

### 🔴 Local Network Protection permission
*(Full details in [networking-and-data-transfer.md](./networking-and-data-transfer.md).)*
- Accessing the **local network** (raw sockets to local IPs, mDNS, SSDP, `NsdManager`,
  unicast/multicast/broadcast to local addresses) now requires permission, surfaced via
  `NEARBY_WIFI_DEVICES`. Rolling out as opt-in first, enforced later.
- **Effect:** casting, device discovery, and local connections **fail** (`EPERM`,
  `ECONNABORTED`) without the grant. DNS to local resolver (port 53) is exempt.

### 🟠 Granular health permissions replace `BODY_SENSORS*`
- `BODY_SENSORS` / `BODY_SENSORS_BACKGROUND` are replaced by `android.permission.health.*`:
  - e.g. `READ_HEART_RATE` (was `BODY_SENSORS`),
    `READ_HEALTH_DATA_IN_BACKGROUND` (was `BODY_SENSORS_BACKGROUND`)
  - New FGS type `FOREGROUND_SERVICE_TYPE_HEALTH`.
- Mobile apps must declare an activity that shows a **privacy policy**, or access is revoked.

```xml
<uses-permission android:name="android.permission.health.READ_HEART_RATE" />
<uses-permission android:name="android.permission.health.READ_HEALTH_DATA_IN_BACKGROUND" />
```

### 🟢 Photo Picker pre-selects app-owned photos
- When the user limits photo access, photos your app created are pre-selected; the user can
  deselect to revoke. No code change required.

---

## Quick migration map (permissions)

| If your app… | You must… | Introduced |
|---|---|---|
| Posts notifications | Request `POST_NOTIFICATIONS`, handle denial | 13 |
| Reads images/video/audio from storage | Use `READ_MEDIA_*` or Photo Picker | 13 |
| Keeps a custom media picker | Add `READ_MEDIA_VISUAL_USER_SELECTED` | 14 |
| Uses full-screen-intent notifications | Gate on `canUseFullScreenIntent()` | 14 |
| Reads heart rate / SpO₂ / temp | Migrate to `health.*` permissions + privacy policy | 16 |
| Talks to devices on the LAN (mDNS, cast, sockets to 192.168.x.x, etc.) | Request `NEARBY_WIFI_DEVICES`, handle denial | 16 |

## Sources

- Android 13: https://developer.android.com/about/versions/13/behavior-changes-13
- Android 14: https://developer.android.com/about/versions/14/behavior-changes-14
- Android 15: https://developer.android.com/about/versions/15/behavior-changes-15
- Android 16: https://developer.android.com/about/versions/16/behavior-changes-16
- Photo Picker: https://developer.android.com/training/data-storage/shared/photopicker
