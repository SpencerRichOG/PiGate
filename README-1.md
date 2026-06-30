# PiGate — Raspberry Pi Wireless Router & Network Monitor

A self-contained wireless router and network monitor built on a Raspberry Pi 3B+. PiGate serves as a fully functional access point with a live stats display, system logging, and a wireless WAN uplink — housed in a custom 3D printed enclosure designed in TinkerCAD.

**Built by [Spencer Rich](https://github.com/SpencerRichOG)**

---

## Photos

> *Photos coming soon — final build with 3D printed enclosure in progress.*

---

## Features

- Wireless access point broadcasting its own network via onboard `wlan0`
- Wireless WAN uplink via USB WiFi 6 adapter (`wlan1`) — no ethernet dependency
- Live 2.8" TFT SPI display showing real-time system metrics (auto-starts on boot)
- System logs offloaded to a dedicated USB drive to protect SD card longevity
- GPIO-controlled cooling fan
- Custom 3D printed enclosure designed from scratch in TinkerCAD

---

## Hardware

| Component | Details |
|---|---|
| SBC | Raspberry Pi 3B+ |
| Display | 2.8" TFT SPI 240x320 V1.0 (ILI9341, capacitive touch) |
| USB WiFi Adapter | BrosTrend AX7L (WiFi 6, AX900, dual-band) |
| Cooling | 5V 2-pin GPIO fan |
| Storage | MicroSD (OS) + USB flash drive (logs) |
| Enclosure | Custom FDM 3D print, designed in TinkerCAD |
| Connection | Female-to-female Dupont jumper wires (2.54mm pitch) |

---

## Network Architecture

```
Internet
   │
   ▼
ISP Router (NachoWifi)
   │
   │  WiFi (wlan1 — BrosTrend AX7L, WAN uplink)
   ▼
Raspberry Pi 3B+ (PiGate)
   │
   │  WiFi (wlan0 — onboard, AP broadcast)
   ▼
Client Devices
```

The Pi bridges its wireless WAN connection (`wlan1`) to its AP interface (`wlan0`), routing traffic between connected clients and the upstream network. Ethernet (`eth0`) is available as a fallback WAN if needed.

---

## Stats Display

The 2.8" TFT displays five real-time metrics, updating every second:

| # | Metric | Source |
|---|---|---|
| 1 | CPU Temperature | `vcgencmd measure_temp` |
| 2 | RAM / Swap Usage | `psutil.virtual_memory()` |
| 3 | Network Throughput (↓/↑) | `psutil.net_io_counters()` |
| 4 | System Uptime | `psutil.boot_time()` |
| 5 | Local IP Address | Socket routing |

The display script runs as a `systemd` service and starts automatically on every boot.

**Display wiring (GPIO to ILI9341 via Dupont jumpers):**

| LCD Pin | Label | Pi Physical Pin |
|---|---|---|
| 1 | VCC | 1 (3.3V) |
| 2 | GND | 6 (GND) |
| 3 | LCD_CS | 24 (GPIO8, CE0) |
| 4 | LCD_RST | 13 (GPIO27) |
| 5 | LCD_RS | 22 (GPIO25) |
| 6 | SDI/MOSI | 19 (GPIO10) |
| 7 | SCK | 23 (GPIO11) |
| 8 | LED | 17 (3.3V, always-on backlight) |
| 9 | SDO/MISO | 21 (GPIO9) |

---

## Software Stack

| Layer | Technology |
|---|---|
| OS | Raspberry Pi OS (Debian Bookworm) |
| Display driver | `luma.lcd` (ILI9341) |
| Display rendering | Python 3 + Pillow |
| System metrics | `psutil` + `vcgencmd` |
| Network management | NetworkManager (`nmcli`) |
| Log management | `rsyslog` |
| Service management | `systemd` |
| Driver installation | BrosTrend installer (DKMS) |

---

## Repository Structure

```
pigate/
├── README.md
├── src/
│   └── stats_display.py       # TFT stats display script
├── config/
│   └── stats_display.service  # systemd service for auto-start
└── docs/
    ├── wiring.md              # GPIO pinout and wiring notes
    ├── network.md             # AP and wireless uplink setup
    └── logging.md             # USB log drive setup
```

---

## Setup Guide

### 1. Enable SPI
```bash
sudo raspi-config
# Interface Options → SPI → Enable
sudo reboot
```

### 2. Install dependencies
```bash
sudo apt install python3-psutil python3-pil python3-luma.lcd fonts-dejavu-core rsyslog
```

### 3. Deploy the display script
```bash
cp src/stats_display.py /home/<user>/stats_display.py
```

### 4. Enable auto-start on boot
```bash
sudo cp config/stats_display.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable stats_display
sudo systemctl start stats_display
```

### 5. Configure wireless uplink
```bash
# Connect USB WiFi adapter to upstream network
sudo nmcli dev wifi connect "<SSID>" password "<password>" ifname wlan1

# Set wlan1 as preferred route
sudo nmcli connection modify "<SSID>" ipv4.route-metric 50
sudo nmcli connection modify "Wired connection 1" ipv4.route-metric 200
sudo nmcli connection modify "<SSID>" connection.autoconnect yes
```

### 6. Set up USB log drive
```bash
sudo mkfs.ext4 /dev/sda1
sudo mkdir /mnt/logs
sudo mount /dev/sda1 /mnt/logs
# Add to /etc/fstab for auto-mount, then configure rsyslog to write to /mnt/logs/syslog
```

---

## Skills Demonstrated

- **Linux system administration** — service configuration, systemd, filesystem management, package management
- **Networking** — access point configuration, DHCP, routing tables, wireless WAN uplink, dual-interface management
- **Python scripting** — SPI display driver integration, system metrics polling, real-time rendering with Pillow
- **Hardware interfacing** — SPI protocol, GPIO wiring, Dupont connector crimping, fan wiring
- **CAD / fabrication** — custom enclosure designed in TinkerCAD, FDM 3D printed
- **Documentation** — technical writeup, pinout tables, network diagrams, setup guides
- **Troubleshooting** — driver installation, kernel module management, routing conflicts, systemd debugging

---

## License

MIT License — see [LICENSE](LICENSE) for details.
