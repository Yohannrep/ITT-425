````markdown
# 🐉 Kali Linux — Set Static IP with `nmcli` (Proxmox NAT)

This guide shows how to set a **static IPv4 address** on a **Kali Linux VM** using **NetworkManager (`nmcli`)** on a **Proxmox NAT bridge**.

---


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

Ahhh okay — you want to set it the **classic network interface file way**, not `nmcli`. Let’s do it.

⚠️ Important first:
Modern Kali uses **NetworkManager by default**, so we either disable it or tell it to leave the interface alone.

---

# 🐉 Set Static IP on Kali (via `/etc/network/interfaces`)

## 1️⃣ Disable NetworkManager control for the interface

Open:

```bash
sudo nano /etc/NetworkManager/NetworkManager.conf
```

Make sure this exists:

```
[ifupdown]
managed=false
```

Save & exit.

---

## 2️⃣ Edit the interfaces file

```bash
sudo nano /etc/network/interfaces
```

Replace everything with something like this:

```bash
auto lo
iface lo inet loopback

auto ens18
iface ens18 inet static
    address 192.168.253.50
    netmask 255.255.255.0
    gateway 192.168.253.1
    dns-nameservers 1.1.1.1 8.8.8.8
```

🔹 Change:

* `ens18` → your interface (`ip a` to confirm)
* IP / gateway → match your network

---

## 3️⃣ Restart networking

```bash
sudo systemctl restart networking
```

If that fails:

```bash
sudo ifdown ens18
sudo ifup ens18
```

---

## 4️⃣ Verify

```bash
ip a
ip route
```

---

# 🧠 Important Notes

If it doesn’t apply:

You may need to **disable NetworkManager completely**:

```bash
sudo systemctl stop NetworkManager
sudo systemctl disable NetworkManager
sudo systemctl restart networking
```

---

# 🚀 Quick Example for Proxmox NAT

```bash
address 192.168.253.50
netmask 255.255.255.0
gateway 192.168.253.1
```

---


