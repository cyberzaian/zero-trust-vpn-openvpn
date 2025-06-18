# 🔐 VPN-Secured RDP Gateway Using OpenVPN on Ubuntu VPS

## 📌 Overview

This project demonstrates how to build a **private, secure RDP gateway** using OpenVPN on an Ubuntu VPS. Only authenticated VPN clients can access the internal Windows RDP server. All traffic flows through a private VPN subnet (`10.8.0.0/24`), with **zero public exposure** of the RDP service.

---

## 🎯 Purpose

To securely access a Windows RDP server via a VPN gateway, ensuring:

- ❌ No public exposure of the Windows Server
- ✅ Only VPN-authenticated users can access RDP
- 🔒 All communications are encrypted end-to-end using OpenVPN

---

## 🖼️ Architecture

```
+------------------------+
|  Your Admin Laptop     |
|  VPN IP: 10.8.0.3      |
+-----------+------------+
            |
       (VPN Tunnel)
            |
+------------------------+    10.8.0.1     +------------------------+
| Ubuntu VPS (OpenVPN)   +----------------+  Windows Server (RDP)   |
| Public IP: x.x.x.x     |                | VPN IP: 10.8.0.2        |
+------------------------+                +------------------------+
```

---

## 🛠️ Installation Steps

### 🔹 1. Install OpenVPN

```bash
wget https://git.io/vpn -O openvpn-install.sh
chmod +x openvpn-install.sh
sudo bash openvpn-install.sh
```

Follow the prompts to set up the OpenVPN server.

---

### 🔹 2. Enable IP Forwarding

```bash
sudo sysctl -w net.ipv4.ip_forward=1
echo "net.ipv4.ip_forward = 1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

---

### 🔹 3. Configure NAT with iptables

Replace `eth0` with your actual public network interface if different:

```bash
# Forward incoming VPN requests
sudo iptables -t nat -A PREROUTING -i eth0 -p udp --dport 1194 -j DNAT --to-destination 10.8.0.1:1194

# Allow traffic forwarding
sudo iptables -A FORWARD -i eth0 -p udp --dport 1194 -d 10.8.0.1 -j ACCEPT

# Enable masquerading for outgoing traffic
sudo iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -o eth0 -j MASQUERADE
```

---

### 🔹 4. Save iptables Rules (Optional)

```bash
sudo apt install iptables-persistent
sudo netfilter-persistent save
```

Or save manually:

```bash
sudo iptables-save > /etc/iptables/rules.v4
```

---

## ✅ Confirm Setup

- Check forwarding rules:
  ```bash
  sudo iptables -t nat -L -n -v
  ```

- Watch for VPN traffic:
  ```bash
  sudo tcpdump -i eth0 udp port 1194
  ```

---

## 👤 Client Creation

After installation, create a VPN client:

```bash
sudo bash openvpn-install.sh
# Choose "Add a new client"
```

Export the `.ovpn` config:

```bash
sudo cp /root/client1.ovpn ~
```

Use this `.ovpn` file to connect from any OpenVPN client.

---
````
IMPORTANT to make the zerotrust access dont forget to install the client in RDP SERVER and Setup the firewall as you need
````
## 🔐 Benefits

- 🔒 **Zero Trust**: No one can reach the RDP server without VPN.
- 🌐 **Private IP Access Only**: RDP works over `10.8.0.x`, not public IP.
- 🧱 **Firewall Friendly**: Prevents port scanning and brute force.
- 📦 **Expandable**: Easily add more clients or internal resources.

---

## 📄 Included

- 📘 `How To Make OpenVPN In Ubuntu VPS Server.pdf`: Full step-by-step guide.

---

## 🙋 About Me

I'm a 19-year-old cybersecurity learner focused on Blue Teaming. This project is part of my journey to building secure infrastructures and growing into a SOC/Defensive security engineer.

---

## 🤝 Let’s Connect

📬 LinkedIn: https://www.linkedin.com/in/zaianahmed  

---

## 📜 License

MIT License
