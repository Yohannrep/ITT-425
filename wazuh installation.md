# 🛡️ Wazuh 4.12 Installation Guide

## Single Node Deployment on Ubuntu 22.04.5 (VM)

This guide walks through installing **Wazuh 4.12** in a **single-node configuration** (Indexer + Manager + Dashboard on one VM).

---

## 💻 VM Requirements (Recommended)

| Resource | Recommended                    | Minimum                |
| -------- | ------------------------------ | ---------------------- |
| CPU      | 4 vCPU                         | 2 vCPU                 |
| RAM      | 8 GB                           | 4 GB (not recommended) |
| Storage  | 100 GB                         | 50 GB                  |
| OS       | Ubuntu Server 22.04.5 (64-bit) | —                      |

---

# 🚀 Installation Steps

---

## ✅ Step 1 — Install Ubuntu Server

1. Create a new VM (Proxmox or preferred hypervisor).
2. Assign:

   * 4 vCPU cores
   * 8 GB RAM
   * 100 GB disk
3. Install **Ubuntu Server 22.04.5 (x86_64)**.
4. During setup:

   * Set hostname (example: `wazuh`)
   * Create user account
   * Install OpenSSH Server

---

## ✅ Step 2 — Update System & Install Dependencies

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install curl nano -y
```

---

## ✅ Step 3 — Install & Enable NGINX

```bash
sudo apt install nginx -y
sudo systemctl start nginx
sudo systemctl enable nginx
```

---

## ✅ Step 4 — Download Wazuh Installer & Config File

```bash
curl -sO https://packages.wazuh.com/4.12/wazuh-install.sh
curl -sO https://packages.wazuh.com/4.12/config.yml
chmod +x wazuh-install.sh
```

---

## ✅ Step 5 — Edit `config.yml`

Open the file:

```bash
nano config.yml
```

Modify it with your **local IP address**:

```yaml
nodes:
  indexer:
    - name: node-1
      ip: 192.168.100.153
  server:
    - name: wazuh-1
      ip: 192.168.100.153
  dashboard:
    - name: dashboard
      ip: 192.168.100.153
```

> ⚠️ Replace `192.168.100.153` with your actual VM IP.

---

## ✅ Step 6 — Generate Configuration Files

```bash
sudo bash wazuh-install.sh --generate-config-files
```

This generates:

```
wazuh-install-files.tar
```

It contains:

* Passwords
* Certificates
* Internal credentials

---

## ✅ Step 7 — Install Wazuh Indexer

```bash
sudo bash wazuh-install.sh --wazuh-indexer node-1
```

---

## ✅ Step 8 — Start the Cluster

⚠️ Run this **only once** after installing the indexer.

```bash
sudo bash wazuh-install.sh --start-cluster
```

---

## ✅ Step 9 — Install Wazuh Server (Manager)

```bash
sudo bash wazuh-install.sh --wazuh-server wazuh-1
```

---

## ✅ Step 10 — Install Wazuh Dashboard

```bash
sudo bash wazuh-install.sh --wazuh-dashboard dashboard
```

---

# 🌐 Access the Dashboard

Open your browser:

```
https://<your-wazuh-ip>/
```

Example:

```
https://192.168.100.153/
```

---

## 🔑 Retrieve Default Admin Password

```bash
tar -axf wazuh-install-files.tar wazuh-install-files/wazuh-passwords.txt -O | grep -P "'admin'" -A 1
```

Username: `admin`
Password: (output from command)

---

# 🧠 Verify Services

Check status:

```bash
sudo systemctl status wazuh-manager
sudo systemctl status wazuh-indexer
sudo systemctl status wazuh-dashboard
```
