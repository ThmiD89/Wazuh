# Week 1 Lab: Installing Ubuntu on VirtualBox for Wazuh

## Learning Outcomes
By the end of this lab, you will be able to:

- Create and configure a virtual machine in VirtualBox  
- Install Ubuntu Server/Desktop manually without unattended automation  
- Configure network settings (Bridged vs. NAT) for host-to-VM access  
- Enable and verify OpenSSH server for remote access  
- Retrieve and document the VM’s IP address for future connections  

## Objective
Set up a fully functional Ubuntu virtual machine in VirtualBox that will serve as the host for the Wazuh SIEM platform. This VM must be accessible from Windows via PuTTY and WinSCP for subsequent security monitoring tasks.

## Scenario
You are a security analyst tasked with deploying an open-source SIEM solution (Wazuh) in a lab environment. Before installing Wazuh, you need a stable Linux base. Your organization uses Windows workstations, so you must configure the Ubuntu VM with proper networking to allow remote management from Windows using SSH and SCP.

## Prerequisites

- [VirtualBox](https://www.virtualbox.org/) installed on Windows/macOS/Linux  
- Ubuntu Server or Desktop ISO (22.04 LTS recommended) – [Download here](https://ubuntu.com/download/server)  
- At least 8 GB of RAM available on your host machine  
- 25 GB of free disk space  

---

## Step 1: Create a New Virtual Machine

1. Open **VirtualBox**.
2. Click **New**.
3. Configure the VM:
   - **Name:** `Ubuntu-Wazuh`
   - **Folder:** your preferred location (e.g., `C:\VMs\`)
   - **ISO Image:** click the dropdown → **Choose a disk file** → select your downloaded Ubuntu ISO
   - **Skip Unattended Installation:** **UNCHECK** this box (we want manual control)
4. Click **Next**.

---

## Step 2: Allocate Hardware Resources

| Setting          | Recommended Value           | Minimum         |
|-----------------|-----------------------------|-----------------|
| **Memory (RAM)** | 8192 MB (8 GB)              | 4096 MB (4 GB)  |
| **CPU Cores**    | 2 (or more if available)    | 1               |
| **Disk Size**    | 25 GB or more               | 20 GB           |

- **Hard disk type:** `VDI (VirtualBox Disk Image)` or `VMDK`
- **Storage on physical hard disk:** `Dynamically allocated`

Click **Next** → **Finish**.

---

## Step 3: Configure Network for Windows Access (Important)

1. Select your `Ubuntu-Wazuh` VM.
2. Click **Settings** → **Network**.
3. **Adapter 1** tab:
   - **Enable Network Adapter:**  checked
   - **Attached to:** choose one:

| Mode        | When to use                                                                 |
|-------------|-----------------------------------------------------------------------------|
| **Bridged** | Windows needs to reach Ubuntu by IP (recommended for PuTTY / WinSCP / Wazuh) |
| **NAT**     | Ubuntu can access internet, but Windows can't reach Ubuntu by default        |

>  If you choose **NAT**, you'll need extra port forwarding to access Ubuntu from Windows.  
> **Bridged** is simpler for this setup.

Click **OK**.

---

## Step 4: Install Ubuntu

1. **Start** the VM.
2. Follow the Ubuntu installer prompts:

   - **Language:** `English`
   - **Keyboard layout:** as preferred
   - **Type of installation:** `Minimal` (or `Ubuntu Server` if using server ISO)

3. **Network setup** (if asked):
   - DHCP is fine – the VM will get an automatic IP

4. **Storage configuration:**
   - Use **Entire disk** (default)

5. **Profile setup:**
   - **Your name:** `Wazuh User` (or any)
   - **Computer name:** `wazuh-server` (or keep default)
   - **Username:** `wazuh-user`
   - **Password:** choose a **strong password** (save it)

6. **SSH setup (very important!):**
   -  **Check** `Install OpenSSH server`
   - This allows PuTTY/WinSCP connections later

7. **Featured server snaps:**  
   - Skip (none needed)

8. Click **Install** and wait for completion.

---

## Step 5: First Boot & Post-Installation

1. When installation finishes, click **Reboot**.
2. The VM will restart.  
   - If asked to remove installation medium, press **Enter**.
3. Log in with your username (`wazuh-user`) and password.

---

## Step 6: Verify Ubuntu is Ready

Inside the Ubuntu VM, run:

```bash
ip a
```

Look for an IP address under `eth0` or `enp0s3`.

Example: `192.168.1.100`

**Save this IP address** – you'll need it to connect via PuTTY/WinSCP and to access the Wazuh dashboard.

---

## Troubleshooting

| Issue                               | Solution                                                                 |
|-------------------------------------|--------------------------------------------------------------------------|
| Can't see IP address in `ip a`      | Run `sudo dhclient` to request an IP from DHCP                          |
| Ubuntu installer doesn't start      | Enable virtualization in BIOS (Intel VT-x / AMD-V)                       |
| Windows can't ping Ubuntu IP        | Change VirtualBox network adapter from NAT to Bridged                    |
| SSH connection refused              | Run `sudo systemctl status ssh` and `sudo ufw allow 22`                  |


Screenshots for helping the installation process :
![image alt](https://github.com/ThmiD89/Wazuh/blob/da4ecbc2ff2c0a710eaecef5293fc6247f30f5a2/Pictures/Screenshot%202026-05-16%20161626.png)
![image alt](https://github.com/ThmiD89/Wazuh/blob/da4ecbc2ff2c0a710eaecef5293fc6247f30f5a2/Pictures/Screenshot%202026-05-16%20170126.png)
![image alt](https://github.com/ThmiD89/Wazuh/blob/da4ecbc2ff2c0a710eaecef5293fc6247f30f5a2/Pictures/Screenshot%202026-05-16%20175850.png)
![image alt](https://github.com/ThmiD89/Wazuh/blob/da4ecbc2ff2c0a710eaecef5293fc6247f30f5a2/Pictures/Screenshot%202026-05-16%20180533.png)
![image alt](https://github.com/ThmiD89/Wazuh/blob/da4ecbc2ff2c0a710eaecef5293fc6247f30f5a2/Pictures/Screenshot%202026-05-16%20180738.png)
![image alt](https://github.com/ThmiD89/Wazuh/blob/da4ecbc2ff2c0a710eaecef5293fc6247f30f5a2/Pictures/Screenshot%202026-05-16%20180931.png)    
![image alt](https://github.com/ThmiD89/Wazuh/blob/da4ecbc2ff2c0a710eaecef5293fc6247f30f5a2/Pictures/Screenshot%202026-05-16%20181008.png)
![image alt](https://github.com/ThmiD89/Wazuh/blob/da4ecbc2ff2c0a710eaecef5293fc6247f30f5a2/Pictures/Screenshot%202026-05-16%20181214.png)
![image alt](https://github.com/ThmiD89/Wazuh/blob/da4ecbc2ff2c0a710eaecef5293fc6247f30f5a2/Pictures/Screenshot%202026-05-16%20181557.png)
![image alt](https://github.com/ThmiD89/Wazuh/blob/da4ecbc2ff2c0a710eaecef5293fc6247f30f5a2/Pictures/Screenshot%202026-05-16%20182822.png)
![image alt](https://github.com/ThmiD89/Wazuh/blob/da4ecbc2ff2c0a710eaecef5293fc6247f30f5a2/Pictures/Screenshot%202026-05-16%20182921.png)
![image alt](https://github.com/ThmiD89/Wazuh/blob/da4ecbc2ff2c0a710eaecef5293fc6247f30f5a2/Pictures/Screenshot%202026-05-16%20183219.png)
![image alt](https://github.com/ThmiD89/Wazuh/blob/da4ecbc2ff2c0a710eaecef5293fc6247f30f5a2/Pictures/Screenshot%202026-05-16%20183340.png)
![image alt](https://github.com/ThmiD89/Wazuh/blob/da4ecbc2ff2c0a710eaecef5293fc6247f30f5a2/Pictures/Screenshot%202026-05-16%20183416.png)
![image alt](https://github.com/ThmiD89/Wazuh/blob/da4ecbc2ff2c0a710eaecef5293fc6247f30f5a2/Pictures/Screenshot%202026-05-16%20184522.png)

