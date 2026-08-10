---
title: "My Android ADB Tuning Guide: What I Changed, Why, and How to Undo It"
description: After setting up Radicale on my Arch Linux machine and facing the dreaded “no internet” exclamation mark (on both Wi‑Fi and mobile data), I decided to take control of my **Xiaomi Poco M3 running LineageOS** using ADB.  Below is a complete log of every `adb shell settings put` command I ran, what it actually does, and – most importantly – how to revert each change to the default Android behaviour.
keywords: Adb,
draft: false
tags:
  - adb
date:
---
> [!danger] This article is ai generated as reference only. 

After setting up Radicale on my Arch Linux machine and facing the dreaded “no internet” exclamation mark (on both Wi‑Fi and mobile data), I decided to take control of my **Android running LineageOS** using ADB.  
Below is a complete log of every `adb shell settings put` command I ran, what it actually does, and – most importantly – how to revert each change to the default Android behaviour.

> ✅ **Prerequisite**  
> USB debugging enabled on the phone, `adb` installed on the computer, and the device authorised.  
> Commands executed from a terminal on my Arch Linux machine.

---

## 1. Connectivity & Captive Portal (The “!” Fix)

These tweaks stop Android from phoning home to Google’s servers to check if the network really has internet.

### Commands executed

```bash
adb shell settings put global captive_portal_mode 0
adb shell settings put global captive_portal_detection_enabled 0
adb shell settings put global captive_portal_use_https 0        # (ran earlier, not shown in log but recommended)
```

### What they do

| Setting | Default | Effect of `0` |
|---------|---------|----------------|
| `captive_portal_mode` | `1` | Completely disable captive portal detection. No more “Sign in to network” pop‑ups, no more exclamation mark. |
| `captive_portal_detection_enabled` | `1` | Older switch; setting to `0` reinforces the above. |
| `captive_portal_use_https` | `1` | Use plain HTTP instead of HTTPS for the connectivity check. Avoids SSL certificate issues on local networks. |

### How to revert

```bash
adb shell settings put global captive_portal_mode 1
adb shell settings put global captive_portal_detection_enabled 1
adb shell settings put global captive_portal_use_https 1
```

After reverting, toggle **Airplane Mode** or reboot to force a fresh network evaluation.

---

## 2. Network & Scanning (Battery + Privacy)

Stop background Wi‑Fi and Bluetooth scanning – even when those radios are off.

### Commands executed

```bash
adb shell settings put global wifi_scan_always_enabled 0
adb shell settings put global ble_scan_always_enabled 0
```

### What they do

| Setting | Default | Effect of `0` |
|---------|---------|----------------|
| `wifi_scan_always_enabled` | `1` | Apps and system can no longer scan for Wi‑Fi networks in the background. Saves battery. |
| `ble_scan_always_enabled` | `1` | Stops Bluetooth Low Energy scanning. Great for privacy and battery. |

### How to revert

```bash
adb shell settings put global wifi_scan_always_enabled 1
adb shell settings put global ble_scan_always_enabled 1
```

> Note: These settings can also be changed graphically in `Settings > Location > Scanning`.

---

## 3. UI Speed & Animations (Make It Snappy)

Turn off all system animations. The phone feels instantly faster.

### Commands executed

```bash
adb shell settings put global transition_animation_scale 0.0
adb shell settings put global window_animation_scale 0.0
adb shell settings put global animator_duration_scale 0.0
```

### What they do

| Setting | Default | Effect of `0.0` |
|---------|---------|----------------|
| `transition_animation_scale` | `1.0` | Disables transition effects between screens. |
| `window_animation_scale` | `1.0` | Disables window open/close animations. |
| `animator_duration_scale` | `1.0` | Disables all other system UI animations. |

### How to revert (restore stock speed)

```bash
adb shell settings put global transition_animation_scale 1.0
adb shell settings put global window_animation_scale 1.0
adb shell settings put global animator_duration_scale 1.0
```

You can also use `0.5x` for a half‑speed compromise.

---

## 4. Privacy & Telemetry Blocking

Prevent Android from sending crash reports, usage statistics, and package verification data to Google.

### Commands executed

```bash
adb shell settings put global send_action_app_error 0
adb shell settings put global upload_debug_log_policy 0
adb shell settings put global package_verifier_enable 0
```

### What they do

| Setting | Default | Effect of `0` |
|---------|---------|----------------|
| `send_action_app_error` | `1` | Stops the “Report to Google” dialog when an app crashes. No more crash logs sent. |
| `upload_debug_log_policy` | `1` | Disables automatic upload of system debug logs (diagnostics). |
| `package_verifier_enable` | `1` | Turns off “Verify apps”. Prevents Google from scanning installed APKs. Slightly improves privacy and reduces background data. |

### How to revert

```bash
adb shell settings put global send_action_app_error 1
adb shell settings put global upload_debug_log_policy 1
adb shell settings put global package_verifier_enable 1
```

---

## 5. Battery & Power Management

Force battery saver mode and clean cache.

### Commands executed

```bash
adb shell settings put global low_power 1
adb shell pm trim-caches 10G
```

### What they do

| Command / Setting | Effect |
|------------------|--------|
| `low_power = 1` | Immediately turns on Android’s built‑in battery saver mode. Equivalent to tapping the battery saver quick tile. |
| `pm trim-caches 10G` | Asks the package manager to free up to 10 GB of cached data from all apps. This is not a permanent setting – it’s a one‑time cleanup. |

### How to revert

- **Battery saver:** turn it off normally from Quick Settings, or run:
  ```bash
  adb shell settings put global low_power 0
  ```
- **Cache trimming:** cannot be undone. The cache will refill over time; no harm done.

---

## 6. Immersive Mode – Hide Status & Navigation Bars

### Command executed

```bash
adb shell settings put global policy_control immersive.full=*
```

### What it does

Forces all apps to run in full‑screen immersive mode – the status bar and navigation bar are hidden until you swipe from the top or bottom edge.

### How to revert

```bash
adb shell settings put global policy_control null
```

This restores the normal system UI behaviour.

---

## 7. Failed Command – Voice Recognition

### Attempted

```bash
adb shell settings put secure voice_recognition_service ""
```

### Result

`Bad arguments` – this command is malformed. Disabling “Hey Google” requires more than an empty string; it’s better done via the Google app settings manually.

### How to properly disable Google hotword detection

On the phone:  
`Google App → Settings → Voice → Voice Match → Turn off "Hey Google"`.

---

## 8. Network Mode (Safety – Not Verified)

I also ran:

```bash
adb shell settings put global preferred_network_mode 9
```

**Warning:** The numeric value `9` is highly device‑ and ROM‑specific. On many LineageOS builds this does **nothing** or may break mobile data. I have not verified its effect on my Poco M3.  
If you ever need to revert, the default value depends on your carrier; a factory reset or `settings delete global preferred_network_mode` is safer.

---

## Quick Reference: Revert Everything in One Script

Save this as `revert_android_tweaks.sh` and run it while the phone is connected via ADB:

```bash
#!/bin/bash
# Revert all the ADB settings changed in this guide

adb shell settings put global captive_portal_mode 1
adb shell settings put global captive_portal_detection_enabled 1
adb shell settings put global captive_portal_use_https 1

adb shell settings put global wifi_scan_always_enabled 1
adb shell settings put global ble_scan_always_enabled 1

adb shell settings put global transition_animation_scale 1.0
adb shell settings put global window_animation_scale 1.0
adb shell settings put global animator_duration_scale 1.0

adb shell settings put global send_action_app_error 1
adb shell settings put global upload_debug_log_policy 1
adb shell settings put global package_verifier_enable 1

adb shell settings put global low_power 0
adb shell settings put global policy_control null

echo "Reverted. Toggle Airplane Mode or reboot for full effect."
```

Make it executable: `chmod +x revert_android_tweaks.sh`  
Run it: `./revert_android_tweaks.sh`

---

## Final Notes

- Most of these changes survive **reboots** and even **OTA updates** (unless the update resets settings).
- If something goes wrong, a **factory reset** from recovery will restore everything to defaults.
- The exclamation mark on my SIM card and Wi‑Fi did **not** disappear immediately after running `captive_portal_mode 0` – I had to toggle **Airplane Mode** once. After that, both icons were clean.

*Last updated: 2026-05-08*  
*Device: Xiaomi Poco M3 / LineageOS*