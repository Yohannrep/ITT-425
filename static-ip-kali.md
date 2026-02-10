````markdown
# 🐉 Kali Linux — Set Static IP with `nmcli` (Proxmox NAT)

This guide shows how to set a **static IPv4 address** on a **Kali Linux VM** using **NetworkManager (`nmcli`)** on a **Proxmox NAT bridge**.

---

## 🧭 Step-by-Step Commands (Copy & Paste Friendly)

### 1️⃣ List NetworkManager connections
```bash
nmcli connection show
````

Look for a connection name like:

```text
Wired connection 1
```

---

### 2️⃣ Bind the connection to the NIC (Proxmox = `ens18`)

```bash
sudo nmcli connection modify "Wired connection 1" connection.interface-name ens18
```

---

### 3️⃣ Set a static IPv4 address (example NAT network)

* Subnet: `192.168.253.0/24`
* Gateway: `192.168.253.1`

```bash
sudo nmcli connection modify "Wired connection 1" \
  ipv4.method manual \
  ipv4.addresses 192.168.253.50/24 \
  ipv4.gateway 192.168.253.1 \
  ipv4.dns "1.1.1.1 8.8.8.8"
```

👉 Replace `192.168.253.50` with any unused IP in your NAT range.

---

### 4️⃣ Restart the network connection

```bash
sudo nmcli connection down "Wired connection 1"
sudo nmcli connection up "Wired connection 1"
```

---

### 5️⃣ Verify the configuration

```bash
ip a
ip route
```

You should see:

* Static IP on `ens18`
* Default route via `192.168.253.1`

---

### 6️⃣ Test connectivity

```bash
ping 192.168.253.1
ping 8.8.8.8
ping google.com
```

---

## 🛠️ Troubleshooting (Only If Needed)

### Interface shows as `unmanaged`

```bash
nmcli device status
sudo nmcli device set ens18 managed yes
sudo systemctl restart NetworkManager
```

---

### Reset NetworkManager connection (last resort)

```bash
sudo nmcli connection delete "Wired connection 1"
sudo nmcli connection add type ethernet ifname ens18 con-name kali-static
```

Reapply the static IP settings using `kali-static`.

---

## ⚡ One-Line Quick Setup (Advanced Users)

```bash
nmcli con mod "Wired connection 1" connection.interface-name ens18 && \
nmcli con mod "Wired connection 1" ipv4.method manual ipv4.addresses 192.168.253.50/24 ipv4.gateway 192.168.253.1 ipv4.dns "1.1.1.1 8.8.8.8" && \
nmcli con down "Wired connection 1" && nmcli con up "Wired connection 1"
```

---

## 🧪 Lab Notes

* NAT = internet access
* Internal bridge = attack/defense traffic
* Dual-NIC Kali is ideal for SOC & pentesting labs

📎 Tested on Kali Linux running on Proxmox VE

```
```
