
# 📁 Mounting Remote Web Server Storage on macOS Using rclone + macFUSE

This guide describes the process of mounting a remote storage (e.g. a WebDAV or cloud server) on macOS using `rclone` and `macFUSE`.

---

## ✅ Requirements

- macOS (tested on Ventura/Sonoma)
- [rclone](https://rclone.org/downloads/) (installed manually, **not via Homebrew**)
- [macFUSE](https://osxfuse.github.io/) (with system extension allowed)

---

## 📦 Installation Steps

### 1. Download and Install rclone (manually)
Do **not** use Homebrew (mount won't work).

- Visit: https://rclone.org/downloads/
- Download the macOS version.
- Extract and move the `rclone` binary to a location in your `$PATH`, e.g.:
  ```bash
  sudo mv rclone /usr/local/bin/
  sudo chmod +x /usr/local/bin/rclone
  ```

---

### 2. Install macFUSE

- Download: https://osxfuse.github.io/
- Install normally.
- After installation, go to:
  - `System Settings` → `Privacy & Security`
  - Scroll to bottom, look for message:
    > *System software from developer "Benjamin Fleischer" was blocked*
  - Click **Allow**
  - Restart your Mac if prompted.

---

## 🔐 3. Configure rclone

Create a remote:
```bash
rclone config
```

Follow prompts to add your remote (e.g. SFTP, WebDAV, Google Drive, etc.).

Example for SFTP:
```
n) New remote
name> mySFTPServer
Storage> sftp
host> example.com
user> username
port> 22
pass> password
key_file> /path/to/private/key
```

> *Remember to add ssh keys to server or use password method*

---

## 📂 4. Create Mount Directory

```bash
mkdir -p ~/mnt/Home-Server
```

---

## 🧷 5. Mount the Remote

```bash
rclone mount myServer:/ ~/mnt/Home-Server
```

- If successful, your remote server appears as a local folder.
- You can browse it via **Finder**:
  - `Finder` → `Go` → `Go to Folder…`
  - Type: `~/mnt/Home-Server`

To make it visible in Finder automatically, you can create a shortcut/alias or mount inside `~/Documents` or `~/Volumes`.

---

## 🛑 6. Unmount

Use:
```bash
umount ~/mnt/Home-Server
```
or
```bash
killall rclone
```

---

## 🚗 7. Setting Up Auto-Run on Startup

### Automatically Mount at Startup (Using a Script)

To ensure that the rclone remote is automatically mounted when your Mac starts, create a script and add it to your startup items.

- Create a Mount Script

Open Terminal and create a new script:

```bash
nano ~/mount_rclone.sh
```
Add the following lines to the file:

```bash
#!/bin/bash
  
REMOTE_NAME="Home-Server:"
MOUNT_POINT="$HOME/mnt/Home-Server"
LOG_FILE="$HOME/bin/logs/rclone-mount.log"

# Create mount point if it doesn't exist
mkdir -p "$MOUNT_POINT"
  
# Check if already mounted 
if mount | grep -q "$MOUNT_POINT"; then
  echo "Already mounted at $MOUNT_POINT"
  exit 0
fi

# Mount
echo "Mounting $REMOTE_NAME to $MOUNT_POINT"
rclone mount "$REMOTE_NAME" "$MOUNT_POINT" \
  --volname "Home-Server" \
  --vfs-cache-mode writes \
  --log-file "$LOG_FILE" \
  --log-level INFO &
```
- Make the Script Executable

Run the following command to make the script executable:

```bash
chmod +x ~/mount_rclone.sh
```
- Add the Script to Startup Items

  - Open System Preferences → Users & Groups.

  - Select the Login Items tab.

  - Click the + button, and add the ~/mount_rclone.sh script.

Now, every time your Mac starts, the remote will automatically mount.

---

## 🧩 Troubleshooting

- **macFUSE extension not loaded**:
  - Open `Privacy & Security`, click “Allow”, and **restart**.
- **Error: "cannot find FUSE"**:
  - Ensure macFUSE is installed and permissions are granted.
- **Error: "no such file or directory"**:
  - Make sure your mount point directory exists before mounting.

---

## 💡 Notes

- `rclone mount` does not support background mounting via Homebrew installs — always use the official binary.
- You can create a script for auto-mounting on login.

---
