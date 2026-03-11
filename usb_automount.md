# Proxmox: USB Disk Automount for Backups

## Overview

Automatically mount a specific USB disk to `/mnt/usb/backup01` when plugged in,
using udev (for hot-plug detection) and a systemd mount unit (for the actual mount).

---

## Prerequisites

For exFAT-formatted disks, install the exFAT utilities:

```bash
apt install exfatprogs
```

---

## Step 1 — Find the Disk's UUID

```bash
lsblk -f /dev/sdX
```

Example output:
```
NAME   FSTYPE FSVER LABEL       UUID
sdb
└─sdb1 exfat  1.0   TSCHER-USBC 1868-3853
```

> **Use the UUID of the partition (`sdb1`), not the disk itself (`sdb`).**
> `/dev/sdb` can change if you plug in other USB devices. UUID is stable.

---

## Step 2 — Create the Mountpoint

```bash
mkdir -p /mnt/usb/backup01
```

---

## Step 3 — Create the systemd Mount Unit

File: `/etc/systemd/system/mnt-usb-backup01.mount`

```ini
[Unit]
Description=USB Backup Disk 01 (TSCHER-USBC)

[Mount]
What=UUID=1868-3853
Where=/mnt/usb/backup01
Type=exfat
Options=defaults,noatime,uid=0,gid=0,umask=022

[Install]
WantedBy=multi-user.target
```

> **Filename must exactly match the mountpoint path**, with `/` replaced by `-`
> and the leading `/` dropped. So `/mnt/usb/backup01` → `mnt-usb-backup01.mount`.

---

## Step 4 — Create the udev Rule

File: `/etc/udev/rules.d/99-usb-backup.rules`

```
ACTION=="add",    ENV{ID_FS_UUID}=="1868-3853", RUN+="/bin/systemctl start mnt-usb-backup01.mount"
ACTION=="remove", ENV{ID_FS_UUID}=="1868-3853", RUN+="/bin/systemctl stop mnt-usb-backup01.mount"
```

This fires when the specific disk (identified by UUID) is plugged in or removed.

---

## Step 5 — Activate

```bash
systemctl daemon-reload
udevadm control --reload-rules
```

---

## Testing

### Check the mount unit is known to systemd:
```bash
systemctl status mnt-usb-backup01.mount
```

### Manually trigger the mount (without unplugging):
```bash
systemctl start mnt-usb-backup01.mount
ls /mnt/usb/backup01
```

### Verify it's mounted:
```bash
mount | grep backup01
```

### Test hot-plug (watch udev events before replugging):
```bash
udevadm monitor --environment
```

---

## How It Works

```
disk plugged in
  → udev rule matches UUID
    → systemctl start mnt-usb-backup01.mount
      → disk appears at /mnt/usb/backup01

disk unplugged
  → udev rule matches UUID
    → systemctl stop mnt-usb-backup01.mount
      → mountpoint goes idle
```

---

## Notes

- **exFAT vs ext4:** exFAT is fine for portability (Windows/Mac compatible).
  If this disk is exclusively for Proxmox backups, reformatting to **ext4** gives
  better permissions handling, journaling, and no extra package dependency.

- **Multiple disks:** For additional USB backup disks, repeat the process with
  different UUIDs and mountpoints (`backup02`, etc.), each getting their own
  `.mount` unit and udev rule line.

- **No `.automount` unit needed** for a simple backup disk. The plain `.mount`
  unit + udev rule is sufficient and easier to reason about.
