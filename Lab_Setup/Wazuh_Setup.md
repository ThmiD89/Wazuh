# Week 3 Lab: Installing Wazuh SIEM on Ubuntu

## Learning Outcomes
By the end of this lab, you will be able to:

- Install Wazuh all-in-one (Manager + Indexer + Dashboard) on Ubuntu  
- Configure Wazuh Indexer (OpenSearch) with secure authentication  
- Set up Wazuh Dashboard with admin credentials  
- Open necessary firewall ports for web access and agent communication  
- Access the Wazuh Dashboard from a Windows browser  
- Verify all Wazuh services are running correctly  

## Objective
Deploy a fully functional Wazuh SIEM platform on the Ubuntu VM created in Week 1. Wazuh will provide security monitoring, intrusion detection, log analysis, and vulnerability detection capabilities. The installation will be accessible via web browser from your Windows machine for ongoing security operations.

## Scenario
Your organization needs an open-source SIEM solution to centralize security event monitoring. You have been tasked with deploying Wazuh in a lab environment before production rollout. The Wazuh server must collect logs from endpoints, provide a web-based dashboard for analysis, and support future agent deployments. You will use PuTTY to run the installation commands and WinSCP to transfer any configuration files if needed.

## Prerequisites

- Ubuntu VM from **Week 1 Lab** (running, with internet access)  
- PuTTY from **Week 2 Lab** (to run commands)  
- Ubuntu VM IP address (e.g., `192.168.1.100`)  
- Ubuntu user with `sudo` privileges (e.g., `wazuh-user`)  
- At least 4 GB RAM free on Ubuntu VM (8 GB recommended)  
- Port 443 accessible from Windows (not blocked by firewall)

---

## Part 1: Prepare Ubuntu for Wazuh Installation

### Step 1: Connect to Ubuntu via PuTTY

1. Open **PuTTY**.
2. Load your saved `Ubuntu-Wazuh` session.
3. Click **Open**.
4. Login with your username and password.

### Step 2: Update System Packages

Run the following commands:

```bash
sudo apt update && sudo apt upgrade -y
```

### Step 3: Install Dependencies

```bash
sudo apt install curl apt-transport-https unzip wget gnupg -y
```

### Step 4: Verify System Requirements

Check available memory:

```bash
free -h
```

Ensure at least **4 GB** (preferably 8 GB) is available.

---

## Part 2: Add Wazuh Repository

### Step 1: Import Wazuh GPG Key

```bash
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | sudo gpg --dearmor | sudo tee /usr/share/keyrings/wazuh.gpg > /dev/null
```

### Step 2: Add Wazuh Repository

```bash
echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt/ stable main" | sudo tee /etc/apt/sources.list.d/wazuh.list
```

### Step 3: Update Package List

```bash
sudo apt update
```

---

## Part 3: Install Wazuh Components (All-in-One)

The all-in-one installation includes three components on the same server:

| Component          | Purpose                                    |
|--------------------|--------------------------------------------|
| **Wazuh Manager**  | Central engine, analyzes logs and alerts   |
| **Wazuh Indexer**  | OpenSearch-based data storage/search       |
| **Wazuh Dashboard**| Web UI for visualization and management    |

### Step 1: Install All Components

```bash
sudo apt install wazuh-manager wazuh-indexer wazuh-dashboard -y
```

### Step 2: Verify Installation

Check that all packages were installed:

```bash
dpkg -l | grep wazuh
```

Expected output shows `wazuh-manager`, `wazuh-indexer`, `wazuh-dashboard`.

---

## Part 4: Configure Wazuh Indexer (OpenSearch)

### Step 1: Generate Password Hash for Admin User

```bash
sudo /usr/share/wazuh-indexer/plugins/opensearch-security/tools/hash.sh
```

You will be prompted to enter a password twice.  
**Copy the generated hash** (starts with `$2y$`). Save it for the next step.

Example output:
```
$2y$12$abcdefghijklmnopqrstuvwxyz1234567890
```

### Step 2: Edit Internal Users Configuration

```bash
sudo nano /etc/wazuh-indexer/opensearch-security/internal_users.yml
```

Find the `admin:` section. Replace the `hash:` line with your generated hash:

```yaml
admin:
  hash: "$2y$12$abcdefghijklmnopqrstuvwxyz1234567890"
  reserved: true
  backend_roles:
  - "admin"
  description: "Admin user"
```

- Save: `Ctrl+O` → `Enter`
- Exit: `Ctrl+X`

### Step 3: Set Correct Ownership

```bash
sudo chown -R wazuh-indexer:wazuh-indexer /etc/wazuh-indexer/
```

### Step 4: Start and Enable Wazuh Indexer

```bash
sudo systemctl daemon-reload
sudo systemctl enable wazuh-indexer
sudo systemctl start wazuh-indexer
```

### Step 5: Verify Indexer Status

```bash
sudo systemctl status wazuh-indexer
```

Look for `active (running)` and `loaded` with no errors.

---

## Part 5: Configure Wazuh Manager

### Step 1: Start and Enable Manager

```bash
sudo systemctl enable wazuh-manager
sudo systemctl start wazuh-manager
```

### Step 2: Verify Manager Status

```bash
sudo systemctl status wazuh-manager
```

Look for `active (running)`.

### Step 3: Test Wazuh API (Optional)

```bash
curl -k -u admin:your_password "https://localhost:55000/"
```

> Note: Dashboard not configured yet, so this may fail until Part 6.

---

## Part 6: Configure Wazuh Dashboard

### Step 1: Edit Dashboard Configuration

```bash
sudo nano /etc/wazuh-dashboard/opensearch_dashboards.yml
```

Add or update these lines at the end of the file:

```yaml
server.host: "0.0.0.0"
opensearch.hosts: ["https://localhost:9200"]
opensearch.ssl.verificationMode: none
opensearch.username: "admin"
opensearch.password: "your_strong_password"
opensearch.requestHeadersWhitelist: ["authorization", "securitytenant"]
opensearch_security.multitenancy.enabled: true
opensearch_security.multitenancy.tenants.preferred: ["Private", "Global"]
server.port: 443
```

- Replace `your_strong_password` with the actual password you used in Part 4.
- Save: `Ctrl+O` → `Enter`
- Exit: `Ctrl+X`

### Step 2: Create SSL Certificate Directory

```bash
sudo mkdir -p /etc/wazuh-dashboard/certs
```

### Step 3: Start and Enable Dashboard

```bash
sudo systemctl enable wazuh-dashboard
sudo systemctl start wazuh-dashboard
```

### Step 4: Verify Dashboard Status

```bash
sudo systemctl status wazuh-dashboard
```

Look for `active (running)`.

---

## Part 7: Configure Firewall (UFW)

### Step 1: Open Required Ports

| Port  | Purpose                          | Protocol |
|-------|----------------------------------|----------|
| 22    | SSH (PuTTY/WinSCP)               | TCP      |
| 443   | Wazuh Dashboard (HTTPS)          | TCP      |
| 55000 | Wazuh API                        | TCP      |
| 1514  | Agent events (TCP)               | TCP      |
| 1515  | Agent enrollment                 | TCP      |
| 1516  | Cluster communication            | TCP      |

```bash
sudo ufw allow 22/tcp
sudo ufw allow 443/tcp
sudo ufw allow 55000/tcp
sudo ufw allow 1514/tcp
sudo ufw allow 1515/tcp
sudo ufw allow 1516/tcp
```

### Step 2: Enable UFW

```bash
sudo ufw enable
```

Type `y` when prompted.

### Step 3: Verify Firewall Rules

```bash
sudo ufw status numbered
```

---

## Part 8: Access Wazuh Dashboard from Windows

### Step 1: Get Ubuntu IP Address

On Ubuntu (via PuTTY):

```bash
ip a | grep inet
```

Example IP: `192.168.1.100`

### Step 2: Open Browser on Windows

Navigate to:

```
https://192.168.1.100
```

>  **Note:** Use `https://` (not `http://`). The port is 443 (default).

### Step 3: Accept Security Warning

- Your browser will show "Your connection is not private" or similar.
- Click **Advanced** → **Proceed to 192.168.1.100 (unsafe)**.
- This is normal because Wazuh uses a self-signed certificate.

### Step 4: Login to Wazuh Dashboard

| Field      | Value                          |
|------------|--------------------------------|
| **Username** | `admin`                        |
| **Password** | The password you set in Part 4 |

### Step 5: Verify Dashboard

You should see:

- **Wazuh welcome screen**
- **Agents summary** (0 agents initially – normal)
- **Modules:** Security Events, Inventory, etc.

 Wazuh is now successfully installed and accessible.

---

## Part 9: Verify All Services (Final Check)

Run these commands from PuTTY to ensure everything is running:

```bash
# Check all three services
sudo systemctl status wazuh-manager --no-pager
sudo systemctl status wazuh-indexer --no-pager
sudo systemctl status wazuh-dashboard --no-pager

# Check listening ports
sudo netstat -tulpn | grep -E '443|55000|1514|1515|9200'
```

Expected output includes:

| Port  | Service           |
|-------|-------------------|
| 443   | wazuh-dashboard   |
| 55000 | wazuh-apid        |
| 1514  | wazuh-manager     |
| 1515  | wazuh-manager     |
| 9200  | wazuh-indexer     |

---

## Troubleshooting

| Issue                                      | Solution                                                                 |
|--------------------------------------------|--------------------------------------------------------------------------|
| Dashboard shows "Unable to connect"        | Wait 2-3 minutes after start → `sudo systemctl restart wazuh-dashboard`  |
| "OpenSearch Security not initialized"      | Run `sudo /usr/share/wazuh-indexer/bin/indexer-security-admin.sh -p`    |
| Port 443 not accessible from Windows       | Check UFW: `sudo ufw allow 443/tcp` and VirtualBox network (Bridged)     |
| Login fails (invalid credentials)          | Reset password hash in `internal_users.yml` and restart indexer          |
| wazuh-indexer fails to start               | Check RAM: `free -h` – need at least 4 GB                                |
| "Connection refused" on dashboard          | Ensure dashboard is running: `sudo systemctl status wazuh-dashboard`     |
| Browser shows "ERR_CONNECTION_REFUSED"     | Verify Ubuntu IP and that port 443 is listening: `sudo ss -tuln \| grep 443` |

---

## Quick Reference Commands

```bash
# Service management
sudo systemctl start|stop|restart|status wazuh-manager
sudo systemctl start|stop|restart|status wazuh-indexer
sudo systemctl start|stop|restart|status wazuh-dashboard

# View logs
sudo journalctl -u wazuh-manager -f
sudo journalctl -u wazuh-indexer -f
sudo journalctl -u wazuh-dashboard -f

# Firewall
sudo ufw status verbose
sudo ufw allow 443/tcp

# Test API (replace password)
curl -k -u admin:your_password "https://localhost:55000/"
```

---

## Summary

| Component          | Status Check Command                        | Access URL (Windows)        |
|--------------------|---------------------------------------------|-----------------------------|
| Wazuh Manager      | `sudo systemctl status wazuh-manager`       | N/A (runs in background)    |
| Wazuh Indexer      | `sudo systemctl status wazuh-indexer`       | `https://<IP>:9200` (API)   |
| Wazuh Dashboard    | `sudo systemctl status wazuh-dashboard`     | `https://<IP>`              |


Let me know if you want me to create **Week 4 Lab (Installing Wazuh Agents)** or **Week 5 Lab (Analyzing Alerts)** in the same format.
