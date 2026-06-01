# Networking & Data Transfer: How apps send and receive data (13 → 16)

How the platform constrains outbound/inbound data movement across the migration. The
headline change for 13 → 16 is the **Local Network Protection permission in Android 16**.

---

## Android 13 (API 33)

### 🟢 Advertising ID gating
- `com.google.android.gms.permission.AD_ID` is required to receive a real advertising ID;
  without it the ID is zeroed. Affects analytics/ads SDKs that send identifiers off-device.

### 🟡 `NEARBY_WIFI_DEVICES` for Wi-Fi APIs
- Wi-Fi discovery/connection APIs can use `NEARBY_WIFI_DEVICES`
  (`neverForLocation`) instead of fine location. (Foreshadows the Android 16 local-network
  change, which reuses this permission.)

---

## Android 14 (API 34)

### 🟡 `ACCESS_NETWORK_STATE` required for networked jobs
- `JobInfo.setRequiredNetworkType()` / `setRequiredNetwork()` now require
  `ACCESS_NETWORK_STATE`, or scheduling throws `SecurityException`. Anything that defers
  uploads/downloads to a network-constrained job is affected.

*(Cleartext/TLS policy is unchanged at 14; the floor is raised at 15 — see below.)*

---

## Android 15 (API 35)

### 🟠 TLS 1.0 and 1.1 are disallowed
- Outbound TLS must be **1.2 or higher**; 1.0/1.1 connections fail.
- **Effect:** any endpoint or pinned client stuck on legacy TLS breaks. Audit third-party
  SDKs and self-hosted/legacy backends. Most public servers already require 1.2+.

> Reminder from earlier baselines: cleartext (HTTP) traffic is blocked by default unless
> permitted via a **Network Security Config**. Prefer HTTPS; scope any cleartext exceptions
> narrowly.

---

## Android 16 (API 36)

### 🔴 Local Network Protection permission — the key 13→16 networking change
- Accessing the **local network** now requires permission (surfaced through
  `NEARBY_WIFI_DEVICES`). Rollout is **opt-in first (~25Q2–26Q2), enforced later**.
- **Requires permission for:**
  - Raw sockets to local IP addresses
  - **mDNS** and **SSDP** discovery
  - `NsdManager` (network service discovery)
  - TCP/UDP to local addresses — unicast, multicast, *and* broadcast
- **What counts as "local network":**
  - IPv4: `169.254.0.0/16`, `100.64.0.0/10`, `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`
  - IPv6: link-local, directly-connected, stub networks, multicast
  - **Excludes** cellular and VPN
- **Exemptions:** DNS to the local resolver (port 53); the Output Switcher casting API.
- **Failure modes without the grant:** socket errors such as `EPERM` / `ECONNABORTED`;
  casting, device discovery, and LAN connections silently stop working.

```xml
<uses-permission android:name="android.permission.NEARBY_WIFI_DEVICES" />
```
```kotlin
// Request at runtime, and degrade gracefully if denied.
if (checkSelfPermission(NEARBY_WIFI_DEVICES) != PERMISSION_GRANTED) {
    requestPermissions(arrayOf(NEARBY_WIFI_DEVICES), REQ)
}
```

Test the enforced behavior before it's mandatory:
```bash
adb shell am compat enable  RESTRICT_LOCAL_NETWORK <pkg>
adb reboot
# ...test discovery/casting/LAN sockets...
adb shell am compat disable RESTRICT_LOCAL_NETWORK <pkg>
```

> **Internet (non-local) traffic is unaffected** by this change — talking to a remote
> server over the internet does **not** require the local-network permission. This control
> is specifically about reaching devices on the user's own LAN.

---

## What this means for an app's data flows (13 → 16)

| Data flow | Constraint by 16 | Action |
|---|---|---|
| HTTPS to a remote backend | TLS ≥ 1.2 (15) | Ensure server/SDKs support 1.2+ |
| HTTP (cleartext) anywhere | Blocked by default | Use HTTPS; narrow Network Security Config exceptions |
| Deferred upload/download via Job | Needs `ACCESS_NETWORK_STATE` (14) | Declare the permission |
| Discover devices on LAN (mDNS/SSDP/NSD) | Needs local-network permission (16) | Request `NEARBY_WIFI_DEVICES`; handle denial |
| Connect to `192.168.x.x` / `10.x.x.x` sockets | Needs local-network permission (16) | Same as above |
| Casting via Output Switcher | Exempt (16) | No change |
| Sending advertising ID | `AD_ID` permission (13) | Declare if using ads/analytics |

## Sources

- Android 16 local network protection: https://developer.android.com/about/versions/16/behavior-changes-16#local-network-protection
- Android 15 TLS: https://developer.android.com/about/versions/15/behavior-changes-15
- Network security config: https://developer.android.com/privacy-and-security/security-config
- JobScheduler network constraints: https://developer.android.com/about/versions/14/behavior-changes-14
