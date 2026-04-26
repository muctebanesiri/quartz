---
title: "Offline Anki Sync: Linux ↔ Android (No Internet Needed)"
description: |-
  Let's say You have Anki on your Linux PC and AnkiDroid on your phone and no internet access, but you want to sync your cards wirelessly.

  You can easily run a local Anki sync server on your machine, turn your PC into a Wi‑Fi hotspot, and connect your phone directly to it.
keywords:
draft: true
tags:
  - Anki
date: 2026-04-26
---
Let's say You have Anki on your Linux PC and AnkiDroid on your phone and no internet access, but you want to sync your cards wirelessly.

You can easily run a local Anki sync server on your machine, turn your PC into a Wi‑Fi hotspot, and connect your phone directly to it.

---

## Step 1: Turn your Arch PC into a hotspot

Make sure your `ap0` interface is up and broadcasting. On Arch, I use `create_ap` or NetworkManager’s hotspot mode. Your PC gets IP `192.168.12.1` – that’s the server address.

## Step 2: Start the *correct* sync server

The built-in `anki --syncserver` is old and broken. Use the modern Python module instead:

```bash
SYNC_USER1=myusername:mypassword python -m anki.syncserver
```

You’ll see:
```
INFO listening addr=0.0.0.0:8080
```

Leave this terminal open.

## Step 3: Configure Anki desktop (on the same PC)

- Open Anki → **Tools → Preferences → Syncing**
- Enable **“Self-Hosted Sync Server”**
- Set URL: `http://127.0.0.1:8080/`
- Now sync – when asked for “AnkiWeb ID”, enter the same username/password you used in `SYNC_USER1`.

## Step 4: Configure AnkiDroid (on your phone)

- Connect your phone to your PC’s hotspot.
- In AnkiDroid: **Settings → Advanced → Custom sync server**
- Enable “Use custom sync server”
- **Sync URL:** `http://192.168.1.1:8080/`
- **Media sync URL:** `http://192.168.1.1:8080/msync/`
- **Username** = your chosen username (e.g., `myusername`)
- **Password** = your chosen password

### ⚠️ Critical Android setting

Google blocks unencrypted HTTP by default. You must allow it for AnkiDroid:

- Go to **Android Settings → Apps → AnkiDroid → Allow cleartext traffic → Yes**

## Step 5: Sync

Tap the sync button in AnkiDroid. The terminal on your Arch PC will show `200 OK` responses – that’s success.

### “Email or password incorrect”?

That error just means you entered credentials in the wrong place. Put your local username/password into AnkiDroid’s **Email address** and **Password** fields – ignore the “AnkiWeb” label.