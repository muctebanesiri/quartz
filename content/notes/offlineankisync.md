---
title: "Offline Anki Sync: Linux ↔ Android (No Internet Needed)"
description: |-
  Let's say You have Anki on your Linux PC and AnkiDroid on your phone and no internet access, but you want to sync your cards wirelessly.

  You can easily run a local Anki sync server on your machine, turn your PC into a Wi‑Fi hotspot, and connect your phone directly to it.
keywords:
draft: false
tags:
  - Anki
date: 2026-04-26
---
Let’s say you have Anki on your Linux PC and AnkiDroid on your phone, no internet access, but you want to sync your cards wirelessly.

You can easily run a local Anki sync server on your machine, turn your PC into a Wi‑Fi hotspot, and connect your phone directly to it.

## Step 1: Turn your Linux PC into a hotspot

Make sure your hotspot interface (e.g., `ap0`) is up and broadcasting. On Arch, I use `create_ap` or NetworkManager’s hotspot mode. Your PC gets an IP like `192.168.12.1` – that’s the server address.

## Step 2: Start the correct sync server (temporary, for testing)

The built-in `anki --syncserver` is old and broken. Use the modern Python module instead:

```bash
SYNC_USER1=myusername:mypassword python -m anki.syncserver
```

You’ll see:

```bash
INFO listening addr=0.0.0.0:8080
```

To use a different port:

```bash
SYNC_PORT=9000 SYNC_USER1=myusername:mypassword python -m anki.syncserver
```

Leave this terminal open while you test. **For a permanent background server, skip to Step 2b.**

## Step 2b: Run the server permanently in the background (systemd)

Instead of keeping a terminal open, create a systemd service so the server starts automatically on boot and restarts if it crashes.

1. **Create a virtual environment and install `anki`** (as your normal user):

   ```bash
   python3 -m venv ~/anki-venv
   source ~/anki-venv/bin/activate
   pip install anki
   deactivate
   ```

2. **Create the data directory**:

   ```bash
   mkdir -p ~/anki-data
   ```

3. **Create the systemd service file**:

   ```bash
   sudo nano /etc/systemd/system/anki-sync-server.service
   ```

   Paste this (replace `myusername` and `mypassword` with your chosen credentials, and adjust the path if your username is different):

   ```ini
   [Unit]
   Description=Anki Sync Server
   After=network.target

   [Service]
   Type=simple
   User=myusername
   Group=myusername
   WorkingDirectory=/home/myusername

   Environment="SYNC_USER1=myusername:mypassword"
   Environment="SYNC_BASE=/home/myusername/anki-data"
   Environment="SYNC_HOST=0.0.0.0"
   Environment="SYNC_PORT=8080"

   ExecStart=/home/myusername/anki-venv/bin/python -m anki.syncserver

   Restart=on-failure
   RestartSec=10

   [Install]
   WantedBy=multi-user.target
   ```

4. **Enable and start the service**:

   ```bash
   sudo systemctl daemon-reload
   sudo systemctl enable --now anki-sync-server.service
   ```

5. **Check that it’s running**:

   ```bash
   sudo systemctl status anki-sync-server.service
   ```

   You should see `active (running)`. The server now runs in the background forever – no terminal needed.

## Step 3: Configure Anki desktop (on the same PC)

- Open Anki → **Tools → Preferences → Syncing**
- Enable **“Self-Hosted Sync Server”**
- Set URL: `http://127.0.0.1:8080/`
- Now sync – when asked for “AnkiWeb ID”, enter the same username/password you used in `SYNC_USER1`.

## Step 4: Configure AnkiDroid (on your phone)

- Connect your phone to your PC’s hotspot.
- In AnkiDroid: **Settings → Advanced → Custom sync server**
- Enable “Use custom sync server”
- **Sync URL:** `http://192.168.12.1:8080/`   (use your PC’s hotspot IP)
- **Media sync URL:** `http://192.168.12.1:8080/msync/`
- **Username** = your chosen username (e.g., `myusername`)
- **Password** = your chosen password

### ⚠️ Critical Android setting

Google blocks unencrypted HTTP by default. You must allow it for AnkiDroid:

- Go to **Android Settings → Apps → AnkiDroid → Allow cleartext traffic → Yes**

## Step 5: Sync

Tap the sync button in AnkiDroid. If you’re using the systemd service, you won’t see live logs – but you can watch them with:

```bash
journalctl -u anki-sync-server.service -f
```

You’ll see `200 OK` responses – that’s success.

### “Email or password incorrect”?

That error just means you entered credentials in the wrong place. Put your local username/password into AnkiDroid’s **Email address** and **Password** fields – ignore the “AnkiWeb” label.