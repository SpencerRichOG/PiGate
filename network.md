# Network Setup

This document covers PiGate's network architecture, access point configuration, and wireless WAN uplink setup.

---

## Architecture Overview

```
Internet
   │
   ▼
ISP Router (upstream network)
   │
   │  WiFi — wlan1 (BrosTrend AX7L, WAN uplink)
   ▼
Raspberry Pi 3B+ (PiGate)
   │
   │  WiFi — wlan0 (onboard, AP broadcast)
   ▼
Client Devices
```

PiGate bridges its wireless WAN connection (`wlan1`) to its AP interface (`wlan0`), routing traffic between connected clients and the upstream network. Ethernet (`eth0`) remains available as a fallback WAN if needed.

---

## Interfaces

| Interface | Role | Description |
|---|---|---|
| `wlan0` | LAN / AP | Onboard Pi WiFi, broadcasts the local access point |
| `wlan1` | WAN | BrosTrend AX7L USB adapter, connects to upstream router |
| `eth0` | Fallback WAN | Ethernet, available as backup — higher route metric than wlan1 |

---

## Routing

NetworkManager manages routing between interfaces. Route metrics determine which interface is preferred — lower metric = higher priority.

| Interface | Route Metric | Priority |
|---|---|---|
| `wlan1` | 50 | Primary WAN (preferred) |
| `eth0` | 200 | Fallback WAN |

To verify the current routing table:
```bash
ip route show
```

Expected output with wlan1 as primary:
```
default via <gateway> dev wlan1 proto dhcp src <ip> metric 50
default via <gateway> dev eth0 proto dhcp src <ip> metric 200
```

---

## Wireless WAN Uplink Setup

### 1. Install USB WiFi adapter driver

BrosTrend provides a one-line installer. Run this while connected via ethernet before plugging in the adapter:

```bash
sudo apt update && sudo apt full-upgrade -y
sudo reboot
```

After reboot, plug in the adapter, then run:
```bash
sh -c 'wget linux.brostrend.com/install -O /tmp/install && sh /tmp/install'
```

Reboot again and verify the adapter is detected:
```bash
ip link show
# Should show wlan1 in the list
```

### 2. Connect to upstream network

```bash
sudo nmcli dev wifi connect "<SSID>" password "<password>" ifname wlan1
```

### 3. Set wlan1 as preferred route

```bash
sudo nmcli connection modify "<SSID>" ipv4.route-metric 50
sudo nmcli connection modify "Wired connection 1" ipv4.route-metric 200
sudo nmcli connection modify "<SSID>" connection.autoconnect yes
sudo nmcli connection up "<SSID>" ifname wlan1
```

### 4. Verify internet flows through wlan1

```bash
ping -I wlan1 -c 4 8.8.8.8
```

### 5. SSH note

When disconnecting ethernet, SSH sessions must be established via the `wlan1` IP address to avoid dropping:

```bash
ssh <user>@<wlan1-ip>
```

Find the wlan1 IP with:
```bash
ip addr show wlan1
```

---

## Access Point (wlan0)

The onboard `wlan0` interface is configured as a local access point using `hostapd` and `dnsmasq`, providing DHCP and DNS to connected clients.

Key configuration files:
- `/etc/hostapd/hostapd.conf` — SSID, password, channel, band
- `/etc/dnsmasq.conf` — DHCP range, DNS upstream
- `/etc/dhcpcd.conf` — static IP for wlan0

---

## Useful Commands

```bash
# Show all interfaces and their state
ip link show

# Show routing table
ip route show

# List available WiFi networks on wlan1
sudo iwlist wlan1 scan | grep ESSID

# Check NetworkManager connection status
nmcli connection show

# Restart NetworkManager
sudo systemctl restart NetworkManager

# Check connected AP clients (if hostapd running)
sudo hostapd_cli all_sta
```
