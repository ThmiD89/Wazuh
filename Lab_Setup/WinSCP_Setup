# Week 2 Lab (Extended): Advanced WinSCP Setup for File Transfers

## Learning Outcomes
By the end of this lab, you will be able to:

- Download and install WinSCP on Windows  
- Establish SCP/SFTP connections to Ubuntu VM  
- Navigate the dual-panel interface (local vs. remote)  
- Transfer files and folders using drag-and-drop  
- Edit remote files directly with built-in or external editors  
- Synchronize directories between Windows and Ubuntu  
- Automate file transfers with WinSCP scripting  
- Troubleshoot common connection and permission issues  

## Objective
Master WinSCP as a secure file transfer tool to manage configuration files, logs, and scripts between your Windows workstation and Ubuntu VM. This lab provides both basic and advanced WinSCP techniques essential for managing Wazuh and other Linux-based security tools.

## Scenario
As a SOC analyst managing a Wazuh server on Ubuntu, you frequently need to:
- Upload custom decoders and rules to `/var/ossec/etc/`
- Download log files for offline analysis
- Edit configuration files using your preferred Windows editor (Notepad++, VS Code)
- Back up important Linux directories to Windows

WinSCP provides a graphical, drag-and-drop solution that is faster and more intuitive than command-line SCP for many tasks.

## Prerequisites

- Windows 10/11 (or older Windows with admin rights)  
- Ubuntu VM from **Week 1 Lab** (running, SSH enabled)  
- Ubuntu VM IP address (e.g., `192.168.1.100`)  
- Ubuntu username and password (e.g., `wazuh-user`)  
- PuTTY (optional – for verification only)  
- Internet connection for downloading WinSCP

---

## Part 1: Download and Install WinSCP

### Step 1: Download the Installer

1. Open a browser on Windows.
2. Go to: [https://winscp.net/](https://winscp.net/)
3. Click the green **Download WinSCP** button.
4. Download the latest **installer** (e.g., `WinSCP-6.3.x-Setup.exe`).

>  **Alternative:** Download the **portable executable** if you don't have admin rights.

### Step 2: Run the Installer

1. Double-click the downloaded `.exe` file.
2. Choose **Typical installation** (recommended for beginners).
3. Select **Commander interface** (dual-panel layout – easier for file transfers).
4. Click **Install**.
5. Click **Finish** when complete.

### Step 3: Launch WinSCP

- From Start Menu, search for `WinSCP` and open it.

---

## Part 2: Create and Save Your First Session

### Step 1: Gather Ubuntu Credentials

From your Ubuntu VM, confirm the IP address:

```bash
ip a | grep inet
```

Example: `192.168.1.100`

### Step 2: Open the Login Dialog

When WinSCP launches, you'll see the **Login** window.

### Step 3: Fill in Connection Details

| Field              | Value                                    |
|--------------------|------------------------------------------|
| **File protocol**  | `SCP` (or `SFTP` – both work)            |
| **Host name**      | `192.168.1.100` (your Ubuntu IP)         |
| **Port number**    | `22`                                     |
| **User name**      | `wazuh-user` (or your Ubuntu username)   |
| **Password**       | Your Ubuntu password                     |

### Step 4: Save the Session

1. Click the **Save** button.
2. In the popup, name the session: `Ubuntu-Wazuh`
3.  Check **Save password** (optional – convenient but less secure).
4. Click **OK**.

### Step 5: Connect

1. Click **Login**.
2. If prompted about the server's host key fingerprint:
   - Click **Yes** or **Accept** (this is the first connection – trust the key).
3. WinSCP will connect and open the main interface.

---

## Part 3: Understanding the WinSCP Interface (Commander Mode)

The main window is split into two panels:

| Panel          | Location                                      |
|----------------|-----------------------------------------------|
| **Left panel**  | Your **Windows** local drives (C:, D:, etc.)  |
| **Right panel** | Your **Ubuntu** remote file system            |

### Key Areas

| Area                     | Purpose                                      |
|--------------------------|----------------------------------------------|
| **Directory tree** (top) | Navigate folders                             |
| **File list** (bottom)   | View files and folders in current directory |
| **Toolbar**              | Quick actions (Copy, Delete, Edit, etc.)    |
| **Command line** (bottom)| Execute remote commands directly             |
| **Transfer queue**       | Shows active file transfers                  |

>  **Tip:** You can swap left/right panels in **View** → **Swap panels** (or press `Ctrl + Shift + S`).

---

## Part 4: Basic File Transfers

### Method 1: Drag and Drop

1. **Navigate** on the left panel to a Windows folder (e.g., `C:\Users\YourName\Desktop`).
2. **Navigate** on the right panel to an Ubuntu folder (e.g., `/home/wazuh-user/`).
3. **Drag a file** from left to right → file uploads to Ubuntu.
4. **Drag a file** from right to left → file downloads to Windows.

### Method 2: Using Buttons

1. **Select** a file or folder in either panel.
2. Click the **Copy** button (or press `F5`).
3. Choose destination and click **OK**.

### Method 3: Context Menu

1. **Right-click** a file or folder.
2. Select **Upload** (to Ubuntu) or **Download** (to Windows).
3. Confirm destination.

### Test Transfer: Create and Upload a Test File

**On Windows side (left panel):**
1. Right-click in empty space → **New** → **Text Document**
2. Name it `test-winscp.txt`
3. Right-click the file → **Edit** (or open in Notepad)
4. Type `Hello from WinSCP!` and save.

**Drag** `test-winscp.txt` to the right panel (Ubuntu home folder).

**Verify via PuTTY:**
```bash
cat /home/wazuh-user/test-winscp.txt
```

Expected output: `Hello from WinSCP!`

 File transfer is working correctly.

---

## Part 5: Editing Remote Files Directly

### Option A: Built-in WinSCP Editor

1. **Right-click** a remote file (e.g., `/home/wazuh-user/test-winscp.txt`).
2. Select **Edit**.
3. WinSCP opens the file in its internal editor.
4. Make changes and click **Save**.
5. The file is automatically uploaded back to Ubuntu.

### Option B: External Editor (Recommended for config files)

1. In WinSCP, go to **Options** → **Preferences**.
2. Navigate to **Editors**.
3. Click **Add** → choose **Notepad++** or **VS Code**.
4. Set as default.
5. Double-click any remote file → opens in your preferred editor.
6. Save → WinSCP prompts to upload changes.

>  **Pro tip:** This is perfect for editing Wazuh configuration files like `/var/ossec/etc/ossec.conf`.

---

## Part 6: Working with Directories (Folders)

### Create a Directory on Ubuntu

1. Navigate to `/home/wazuh-user/` on the right panel.
2. Press `F7` or click **New Folder** toolbar button.
3. Name it `winscp-test-folder`.
4. Click **OK**.

### Upload an Entire Folder

1. On the left panel (Windows), create a folder with a few test files.
2. **Drag the entire folder** from left to right panel.
3. WinSCP uploads all files and subfolders recursively.

### Download an Entire Folder

1. On the right panel (Ubuntu), select a folder.
2. **Drag it** to the left panel (Windows).
3. WinSCP downloads all contents.

---

## Part 7: Synchronization (Advanced)

Sync allows you to mirror directories between Windows and Ubuntu.

### Step 1: Prepare Test Folders

- **Windows:** `C:\WinSCP-Sync-Test` (create a few files)
- **Ubuntu:** `/home/wazuh-user/sync-test` (empty)

### Step 2: Run Synchronization

1. In WinSCP, navigate to **Commands** → **Synchronize**.
2. Choose **Synchronization type**:

| Type              | Behavior                                      |
|-------------------|-----------------------------------------------|
| **Local → Remote**| Push Windows changes to Ubuntu (upload)       |
| **Remote → Local**| Pull Ubuntu changes to Windows (download)     |
| **Both**          | Two-way sync (careful – can overwrite)        |

3. Select **Local directory** (Windows) and **Remote directory** (Ubuntu).
4. Check **Preview changes** first.
5. Click **OK** → WinSCP shows what will be transferred.
6. Click **Synchronize** to execute.

---

## Part 8: Using WinSCP Command Line (Within the Interface)

At the bottom of the WinSCP window, there's a **command line** panel.

### Basic Remote Commands

```bash
# List files in current directory
ls -la

# Change directory
cd /var/log

# Show current directory
pwd

# Create a directory
mkdir new-folder

# Remove a file
rm test.txt

# View file content (truncated)
cat /etc/hostname
```

>  **Note:** These commands run on the **Ubuntu** side, not Windows.

---

## Part 9: Bookmarks (Quick Navigation)

### Create a Bookmark

1. Navigate to a frequently used Ubuntu folder (e.g., `/var/ossec/etc/`).
2. Go to **Bookmarks** → **Add Bookmark**.
3. Name it `Wazuh Config`.
4. Click **OK**.

### Use a Bookmark

- Click the **Bookmarks** dropdown in the toolbar.
- Select `Wazuh Config` → instantly navigates there.

---

## Part 10: Security Best Practices

### Option A: Use SSH Keys Instead of Passwords

1. In PuTTY, generate a key pair using `puttygen.exe`.
2. Copy the public key to Ubuntu:
   ```bash
   mkdir -p ~/.ssh
   echo "ssh-rsa AAAAB3..." >> ~/.ssh/authorized_keys
   chmod 600 ~/.ssh/authorized_keys
   ```
3. In WinSCP, load your session → **Edit** → **Authentication**.
4. Browse to your `.ppk` private key file.
5. Save and connect – no password required.

### Option B: Disable Saving Passwords

- When saving a session, **uncheck** `Save password`.
- WinSCP will prompt for password each time.

---

## Part 11: Scripting WinSCP (For Automation)

Create a script to automate file transfers.

### Step 1: Create a Script File

In Windows, create `upload-logs.txt`:

```
open scp://wazuh-user:password@192.168.1.22/ -hostkey="*"
cd /var/ossec/logs
lcd C:\Logs\
get *.log
exit
```

### Step 2: Run the Script

From Windows command prompt:

```bash
winscp.com /script=upload-logs.txt
```

>  Useful for scheduled backups using Windows Task Scheduler.

---

## Troubleshooting

| Issue                                      | Solution                                                                 |
|--------------------------------------------|--------------------------------------------------------------------------|
| "Connection refused"                       | SSH not running on Ubuntu → `sudo systemctl enable ssh --now`            |
| "Host does not exist"                      | Wrong IP address → verify with `ip a` on Ubuntu                          |
| "Authentication failed"                    | Wrong username/password or SSH key not configured                        |
| "Permission denied" when uploading         | Trying to write to system folder → use `/home/wazuh-user/` or `sudo`     |
| Files transfer very slowly                 | Switch from SCP to SFTP (sometimes faster over high latency)             |
| "Host key fingerprint mismatch"            | Ubuntu OS reinstalled → delete saved host key in WinSCP Preferences      |
| Can't see hidden files (starting with .)   | Go to **View** → **Show Hidden Files**                                   |
| Drag and drop not working                  | Try copy/paste (`Ctrl+C` / `Ctrl+V`) instead                             |
| Remote file opens as read-only             | File permissions issue → `sudo chmod 666 filename` on Ubuntu (temporary) |

---

## Quick Reference Shortcuts

| Action                          | Shortcut                |
|--------------------------------|-------------------------|
| Copy selected files            | `F5`                    |
| Move selected files            | `F6`                    |
| Delete selected files          | `Delete` or `F8`        |
| Create new folder              | `F7`                    |
| Rename file/folder             | `F2`                    |
| Edit remote file               | `F4`                    |
| Refresh file list              | `Ctrl + R`              |
| Open terminal (PuTTY)          | `Ctrl + P` (if installed) |
| Swap left/right panels         | `Ctrl + Shift + S`      |
| Go to parent directory         | `Backspace`             |
| Command line focus             | `Ctrl + T`              |

---

## Summary

| Task                          | Method                                          |
|-------------------------------|-------------------------------------------------|
| Upload file to Ubuntu         | Drag from left → right panel                    |
| Download file from Ubuntu     | Drag from right → left panel                    |
| Edit remote config file       | Double-click → external editor → save           |
| Sync folders                  | Commands → Synchronize                          |
| Navigate quickly              | Bookmarks                                       |
| Automate transfers            | WinSCP script + Task Scheduler                  |


