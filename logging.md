# Logging Setup

This document covers PiGate's system logging configuration, which offloads log writes from the SD card to a dedicated USB flash drive to extend SD card lifespan.

---

## Why Offload Logs?

A Raspberry Pi running 24/7 as a router writes system logs constantly. SD cards have a limited number of write cycles and are the most common failure point on always-on Pi setups. Writing logs to a separate USB drive eliminates this wear on the SD card entirely.

---

## Hardware

| Component | Details |
|---|---|
| USB drive | ~4GB flash drive, formatted ext4 |
| Mount point | `/mnt/logs` |
| Device path | `/dev/sda1` |

---

## Setup

### 1. Format the USB drive

Plug the drive into the Pi, then identify its device name:

```bash
lsblk
```

The USB drive will appear as `sda` (with partition `sda1`). Format it as ext4:

```bash
sudo umount /dev/sda1   # ignore "not mounted" errors
sudo mkfs.ext4 /dev/sda1
```

Note the UUID from the mkfs output — you'll need it for fstab.

### 2. Mount the drive

```bash
sudo mkdir /mnt/logs
sudo mount /dev/sda1 /mnt/logs
```

Verify it mounted:
```bash
df -h | grep sda
```

### 3. Configure auto-mount on boot

Add the drive to `/etc/fstab` using its UUID (replace with your actual UUID from the mkfs output):

```bash
sudo nano /etc/fstab
```

Add this line at the bottom:
```
UUID=<your-uuid-here>  /mnt/logs  ext4  defaults,noatime  0  2
```

The `noatime` option prevents the filesystem from recording access times on every file read, further reducing unnecessary writes.

Test the fstab entry without rebooting:
```bash
sudo mount -a
```

No output means success.

### 4. Install and configure rsyslog

```bash
sudo apt install rsyslog
```

Edit the rsyslog config to write logs to the USB drive:

```bash
sudo nano /etc/rsyslog.conf
```

Add this line at the bottom:
```
*.*    /mnt/logs/syslog
```

Create the log file and set permissions:
```bash
sudo touch /mnt/logs/syslog
sudo chown root:root /mnt/logs/syslog
sudo chmod 644 /mnt/logs/syslog
```

Restart rsyslog:
```bash
sudo systemctl restart rsyslog
```

### 5. Verify logs are writing

```bash
tail /mnt/logs/syslog
```

You should see recent system events. After a reboot, fresh boot entries confirm auto-mount and rsyslog are both working correctly.

---

## Log Contents

The syslog captures all system-level events including:

- Boot and shutdown sequences
- systemd service start/stop events (including the stats display service)
- Network interface changes (DHCP, routing updates)
- SSH login/logout events
- sudo usage
- Hardware events (USB device attach/detach)

---

## Useful Commands

```bash
# View recent log entries
tail /mnt/logs/syslog

# Follow log in real time
tail -f /mnt/logs/syslog

# Search logs for a specific service
grep "stats_display" /mnt/logs/syslog

# Check USB drive usage
df -h /mnt/logs

# Check drive is mounted
mount | grep /mnt/logs

# Verify fstab entry
cat /etc/fstab
```

---

## Notes

- The USB drive UUID is used in fstab instead of the device path (`/dev/sda1`) — this ensures the correct drive is always mounted even if the device name changes between reboots
- Log rotation is not configured in this build — for long-term deployments, consider adding `logrotate` to prevent the syslog file from growing indefinitely
- The drive has ~3.5GB of usable space, which is sufficient for years of typical system log volume
