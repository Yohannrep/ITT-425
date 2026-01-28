Bet 😎 — here’s **ONE BIG, CLEAN, DROP-IN Markdown** that replaces the Word doc **1:1**, but looks *chef’s kiss* on GitHub.
You can paste this straight into a `README.md` or any `.md` file and be done.

---

````markdown
# Proxmox NAT Bridge (nat8)
**Subnet:** `192.168.253.0/24`  
**Gateway:** `192.168.253.1`  
**Bridge Name:** `nat8`  
**VM Interface:** `ens18`  

**Use Case:**  
SOC labs • DFIR environments • Malware sandboxes • Internal-only VM networks

---

## 📘 Overview

This document provides a complete, step-by-step guide for creating a **NAT-only
bridge** named `nat8` on **Proxmox VE** and configuring an **Ubuntu virtual
machine with a static IP address** using Netplan.

Using NAT allows virtual machines to:
- Access the internet for updates and tooling
- Remain isolated from the physical LAN
- Prevent inbound connections unless explicitly forwarded

This setup is ideal for **Security Operations Center (SOC)** labs, malware
analysis environments, and cybersecurity coursework.

---

## 🧱 Part 1: Proxmox NAT Bridge Configuration

### Prerequisites
- Proxmox VE host with a working outbound bridge (typically `vmbr0`)
- Root access to the Proxmox host
- `ifupdown2` installed (default on Proxmox)

---

### Step 1: Access the Proxmox Shell
SSH into the Proxmox host or use the web console:

```bash
ssh root@<proxmox-ip>
````

---

### Step 2: Edit the Network Configuration

```bash
nano /etc/network/interfaces
```

---

### Step 3: Create the `nat8` Bridge

Add the following configuration **below your existing `vmbr0` configuration**.
Do **not** remove or modify existing bridges.

```ini
auto nat8
iface nat8 inet static
    address 192.168.253.1/24
    bridge_ports none
    bridge_stp off
    bridge_fd 0

    post-up   echo 1 > /proc/sys/net/ipv4/ip_forward
    post-up   iptables -t nat -A POSTROUTING -s 192.168.253.0/24 -o vmbr0 -j MASQUERADE
    post-down iptables -t nat -D POSTROUTING -s 192.168.253.0/24 -o vmbr0 -j MASQUERADE
```

#### What this does:

* Creates an internal bridge (`nat8`)
* Assigns the Proxmox host as the gateway (`192.168.253.1`)
* Enables IPv4 forwarding
* NATs traffic out through `vmbr0`

> ⚠️ If your outbound bridge is not `vmbr0`, find it using:
>
> ```bash
> ip route | grep default
> ```

---

### Step 4: Apply Network Changes Safely

```bash
ifreload -a
```

This applies changes without dropping network connectivity.

---

## 🖧 Part 2: Attach a VM to `nat8`

In the Proxmox web UI:

1. Select the VM
2. Go to **Hardware**
3. Add or edit **Network Device**
4. Configure:

   * **Bridge:** `nat8`
   * **Model:** `VirtIO`
   * **Firewall:** optional

---

## 🐧 Part 3: Ubuntu Static IP Configuration

### Environment

* Ubuntu Server or Desktop
* Netplan (systemd-networkd)
* Interface name: `ens18`

---

### Step 1: Edit Netplan Configuration

```bash
sudo nano /etc/netplan/50-cloud-init.yaml
```

> ⚠️ Cloud-init manages this file. Editing another Netplan file may be overwritten
> on reboot.

---

### Step 2: Configure Static IP

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    ens18:
      dhcp4: no
      addresses:
        - 192.168.253.10/24
      gateway4: 192.168.253.1
      nameservers:
        addresses:
          - 1.1.1.1
          - 8.8.8.8
```

---

### Step 3: Apply Netplan

```bash
sudo netplan apply
```

---

## 📌 IP Addressing Scheme

| Role / Service | IP Address        |
| -------------- | ----------------- |
| NAT Gateway    | `192.168.253.1`   |
| TheHive        | `192.168.253.10`  |
| Wazuh          | `192.168.253.11`  |
| Shuffle        | `192.168.253.12`  |
| Sandbox VMs    | `192.168.253.20+` |

---

## ✅ Validation & Testing

Inside the Ubuntu VM:

```bash
ping 8.8.8.8
ping google.com
```

Expected results:

* IP ping succeeds → NAT is working
* DNS ping succeeds → name resolution is working

---

## 🔐 Security Notes

* This network is **isolated by default**
* No inbound access unless **explicit port forwarding** is configured
* Ideal for:

  * Malware detonation
  * DFIR tooling
  * SOC training labs
  * Student environments

---
