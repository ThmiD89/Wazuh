# Week 2 Lab: Configuring PuTTY and WinSCP on Windows for Ubuntu Access

## Learning Outcomes
By the end of this lab, you will be able to:

- Install PuTTY and WinSCP on Windows  
- Establish SSH connections to Ubuntu VM using PuTTY  
- Transfer files between Windows and Ubuntu using WinSCP (SCP protocol)  
- Save and manage session profiles for repeated access  
- Navigate the Ubuntu file system remotely from Windows  

## Objective
Configure Windows-based remote access tools (PuTTY for terminal, WinSCP for file transfer) to connect securely to the Ubuntu VM created in Week 1. These tools will be essential for managing the Wazuh installation and transferring log files.

## Scenario
Your Ubuntu VM is running headlessly (no GUI) or you prefer working from your Windows workstation. As a SOC analyst, you need efficient command-line access to the Linux server (via SSH) and the ability to transfer configuration files, scripts, and logs between Windows and Ubuntu. PuTTY provides the terminal, while WinSCP offers a drag-and-drop file interface.

## Prerequisites

- Windows 10/11 (or older Windows with admin rights)  
- Ubuntu VM from **Week 1 Lab** running and powered on  
- Ubuntu VM IP address (from `ip a` command)  
- Ubuntu username and password (e.g., `wazuh-user`)  
- SSH enabled on Ubuntu (verified with `sudo systemctl status ssh`)

---

## Part 1: Install PuTTY

### Step 1: Download PuTTY

![image alt](https://github.com/ThmiD89/Wazuh/blob/6821197e387757ad92842ae809c64b717ddc34b9/Pictures/Screenshot%202026-05-17%20140310.png)

1. Open a browser on Windows.
2. Go to: [https://www.putty.org/](https://www.putty.org/)
3. Click the **Download PuTTY** link.
4. Download the **64-bit MSI installer** (e.g., `putty-64bit-installer.msi`) or the standalone `putty.exe`.

>  **Recommended:** Use the MSI installer for better integration.

### Step 2: Install PuTTY

1. Run the downloaded `.msi` file.
2. Click **Next** → accept the license → **Next**.
3. Choose installation directory (default is fine).
4. Click **Install** → **Finish**.

### Step 3: Launch PuTTY

- From Start Menu, search for `PuTTY` and open it.

---

## Part 2: Connect to Ubuntu via PuTTY

### Step 1: Gather Ubuntu VM IP Address

On your Ubuntu VM, run:

```bash
ip a
```

Look for `inet` under `eth0` or `enp0s3`.  
Example IP: `192.168.1.100`

Keep this IP handy.

![image alt](https://github.com/ThmiD89/Wazuh/blob/a9c35f13b8097a3f069e882ff95a9f62b3e6403c/Pictures/Screenshot%202026-05-17%20140727.png)

### Step 2: Configure PuTTY Session

![image alt](

1. In PuTTY, set the following:

| Field           | Value                          |
|----------------|--------------------------------|
| **Host Name**   | `192.168.1.100` (your Ubuntu IP) |
| **Port**        | `22`                           |
| **Connection type** | `SSH`                      |

2. **Save the session (optional but recommended):**
   - Under **Saved Sessions**, type: `Ubuntu-Wazuh`
   - Click **Save**

### Step 3: Connect

1. Click **Open**.
2. First connection – click **Accept** (SSH security alert).
3. Login as:
   - **Username:** `wazuh-user` (or your Ubuntu username)
   - **Password:** (type it – no characters will appear, this is normal)

### Step 4: Verify Connection

Once logged in, you should see the Ubuntu command prompt:

```bash
wazuh-user@wazuh-server:~$
```

Run a test command:

```bash
whoami
# Should return: wazuh-user

pwd
# Should show: /home/wazuh-user
```

You are now remotely controlling your Ubuntu VM from Windows.

---

## Part 3: Install WinSCP

### Step 1: Download WinSCP

1. Open a browser on Windows.
2. Go to: [https://winscp.net/](https://winscp.net/)
3. Click **Download WinSCP**.
4. Download the latest **installer** (e.g., `WinSCP-6.x.x-Setup.exe`).

### Step 2: Install WinSCP

1. Run the installer.
2. Select **Typical installation**.
3. Choose **Commander interface** (recommended for beginners).
4. Click **Install** → **Finish**.

---

## Part 4: Transfer Files with WinSCP

### Step 1: Create a New Session

1. Open **WinSCP**.
2. In the login window, configure:

| Field           | Value                          |
|----------------|--------------------------------|
| **File protocol** | `SCP` (or SFTP – both work)  |
| **Host name**   | `192.168.1.100` (your Ubuntu IP) |
| **Port number** | `22`                           |
| **User name**   | `wazuh-user`                   |
| **Password**    | your Ubuntu password           |

### Step 2: Save Session

1. Click **Save**.
2. Name the session: `Ubuntu-Wazuh`
3. Check **Save password** (optional, for convenience).
4. Click **OK**.

### Step 3: Connect

1. Click **Login**.
2. If prompted about the server's host key, click **Yes** or **Accept**.

### Step 4: Explore the Interface

You'll see two panels:

| Panel          | Shows                                      |
|----------------|--------------------------------------------|
| **Left**       | Your Windows local drives (C:, D:, etc.)   |
| **Right**      | Ubuntu VM's file system                    |

### Step 5: Transfer a Test File

1. On the **Windows side** (left panel), create a test file:
   - Right-click → New → Text Document → name it `test.txt`
2. **Drag and drop** `test.txt` from left to right panel.
3. On the **Ubuntu side** (right panel), verify the file appears.

### Step 6: Verify via PuTTY

Switch to PuTTY and run:

```bash
ls -la /home/wazuh-user/
```

You should see `test.txt` listed.

File transfer is working correctly.

---

## Part 5: Advanced PuTTY Settings (Optional but Useful)

### Save Session Output to a Log File

1. In PuTTY, go to **Category** → **Logging**.
2. Select **All session output**.
3. Choose a log file path (e.g., `C:\Logs\putty-session.log`).
4. Save the session.

### Change Font Size

1. **Category** → **Window** → **Appearance** → **Font settings**.
2. Click **Change** → select a larger font (e.g., Consolas, 14pt).

### Copy & Paste in PuTTY

| Action          | Method                                      |
|----------------|---------------------------------------------|
| **Copy**        | Select text with mouse (automatically copies) |
| **Paste**       | Right-click anywhere in the PuTTY window    |

---

## Troubleshooting

| Issue                               | Solution                                                                 |
|-------------------------------------|--------------------------------------------------------------------------|
| "Network error: Connection refused" | SSH not running on Ubuntu → `sudo systemctl enable ssh --now`            |
| "Host does not exist"               | Wrong IP address → verify with `ip a` on Ubuntu                          |
| PuTTY closes immediately            | Wrong username/password → check credentials                              |
| WinSCP "Authentication failed"      | Password incorrect or SSH service not running                            |
| Can't ping Ubuntu from Windows      | Change VirtualBox network to **Bridged** (Week 1 Lab, Step 3)            |
| WinSCP permission denied            | You're trying to write to a system folder → use `/home/wazuh-user/`      |

---

## Quick Reference Commands (Run from PuTTY)

```bash
# Check your username
whoami

# Show current directory
pwd

# List files with details
ls -la

# Create a directory
mkdir test-folder

# Remove a file
rm test.txt

# Check Ubuntu IP address
ip a | grep inet

# Restart SSH service if needed
sudo systemctl restart ssh
```



## Summary

| Tool      | Purpose                          | Connection Info                          |
|-----------|----------------------------------|------------------------------------------|
| **PuTTY** | Command-line terminal via SSH    | IP:Port 22 → username/password           |
| **WinSCP**| File transfer (GUI drag-and-drop)| SCP/SFTP protocol, same credentials      |





This follows the exact same structure as your screenshot – with **Learning Outcomes**, **Objective**, **Scenario**, **Prerequisites**, broken into **Parts**, tables, code blocks, and a **Troubleshooting** section. Let me know if you want the **Wazuh installation lab** in the same format.
