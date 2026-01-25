# Configure a Static IP on Ubuntu SOC VMs

*(TheHive, Wazuh, Shuffle)*

All SOC virtual machines **must use a static IP address** to ensure consistent access to services and prevent breakage after reboots.

These instructions apply to **Ubuntu VMs deployed in Proxmox** and use **Netplan with `systemd-networkd`**.

---

## Step 1: Identify the Network Interface

Run:

```bash
ip -c a
```

Typical interface names in Proxmox include:

* `ens18`
* `ens33`
* `eth0`

**Example used in this guide:**

```text
ens33
```

---

## Step 2: Locate the Netplan Configuration File

List available Netplan files:

```bash
ls /etc/netplan/
```

You will usually see:

```text
01-netcfg.yaml
50-cloud-init.yaml
```

> ⚠️ **Important**
> If `50-cloud-init.yaml` exists, **edit that file**.
> Cloud-init will overwrite other Netplan files on reboot.

---

## Step 3: Edit the Netplan Configuration

Open the Netplan file:

```bash
sudo nano /etc/netplan/50-cloud-init.yaml
```

Use the following **standard SOC VM template**:

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    ens33:
      dhcp4: no
      addresses:
        - 192.168.253.142/24
      gateway4: 192.168.253.2
      nameservers:
        addresses:
          - 8.8.8.8
          - 1.1.1.1
```

---

## Step 4: Adjust for Each SOC VM

Change **only the IP address** for each VM.

| VM Name | Example Static IP |
| ------- | ----------------- |
| TheHive | `192.168.253.142` |
| Wazuh   | `192.168.253.143` |
| Shuffle | `192.168.253.144` |

All SOC VMs share:

* Same subnet (`/24`)
* Same gateway
* Same DNS servers

---

## Step 5: Apply the Configuration

Apply safely (recommended when using SSH):

```bash
sudo netplan try
```

If successful, apply permanently:

```bash
sudo netplan apply
```

---

## Step 6: Verify Networking

Check routing:

```bash
ip route
```

**Expected output example:**

```text
default via 192.168.253.2 dev ens33 proto static
192.168.253.0/24 dev ens33 proto kernel scope link src 192.168.253.142
```

Test external connectivity:

```bash
ping -c 4 8.8.8.8
```

---

## ⚠️ Common Mistakes to Avoid

* ❌ Editing `01-netcfg.yaml` when `50-cloud-init.yaml` exists
* ❌ Using an IP address inside the DHCP pool
* ❌ Incorrect interface name (`ens33` ≠ `ens18`)
* ❌ YAML indentation errors (spaces matter)

---

## 📌 Why Static IPs Are Required

* SOC services rely on **fixed endpoints**
* Docker containers map to known host IPs
* Lab instructions and screenshots remain consistent
* Prevents network changes after reboots
