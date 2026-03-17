
# Raspberry Pi Appliance Hardening
## Long-term SD card reliability without overlay filesystem

**Goal:** Minimize SD card writes on a headless appliance (e.g. paging system, kiosk, sensor node) while keeping the root filesystem writable and the system easy to maintain and update.

**Tested on:** Raspberry Pi OS Lite (Bookworm / Debian 12)

---

## 1. Start from a vanilla Raspberry Pi OS Lite

Use the Raspberry Pi Imager to flash the image. In the imager's advanced settings (gear icon), pre-configure:

- Hostname
- SSH enabled
- WiFi credentials (if needed)
- Username / password

Boot, SSH in, then:

```bash
sudo apt update && sudo apt full-upgrade -y
sudo reboot
```

---

## 2. Disable swap

no need - raspibian doesn´t swap to ssd.

## 3. Mount volatile directories as tmpfs

Edit `/etc/fstab` and add the following lines:

```
tmpfs   /tmp            tmpfs   defaults,noatime,nosuid,size=50m    0 0
tmpfs   /var/tmp        tmpfs   defaults,noatime,nosuid,size=30m    0 0
tmpfs   /var/log        tmpfs   defaults,noatime,nosuid,size=50m    0 0
tmpfs   /var/spool      tmpfs   defaults,noatime,nosuid,size=10m    0 0
```

Also add `noatime` to the root filesystem line (reduces access-time writes). It will look something like this:

```
PARTUUID=xxxxxxxx-02  /  ext4  defaults,noatime  0  1
```

Apply without rebooting:

```bash
sudo systemctl daemon-reload
sudo mount -a
```

> **Note:** `/var/log` in RAM means logs are lost on reboot. For an appliance that runs forever, this is fine. If you need post-crash diagnostics, see the optional section at the end.

---

## 4. Configure systemd journal to RAM only

By default, journald may write to disk. Force it to stay in RAM:

Edit `/etc/systemd/journald.conf`:

```ini
[Journal]
Storage=volatile
RuntimeMaxUse=20M
```

Restart the journal:

```bash
sudo systemctl restart systemd-journald
```

Verify:

```bash
journalctl --disk-usage   # Should show usage in /run, not /var/log/journal
```

---

## 5. Disable unnecessary services

Fewer running services = fewer writes and less RAM usage. Common candidates on a headless appliance:

```bash
# Disable Bluetooth if not needed
sudo systemctl disable bluetooth hciuart

# Disable WiFi power management (causes connection drops)
sudo iwconfig wlan0 power off
# Make it persistent:
echo 'iwconfig wlan0 power off' | sudo tee /etc/rc.local
# Or via a systemd service / /etc/network/interfaces depending on your setup

# Disable triggerhappy (GPIO button daemon, not needed on most appliances)
sudo systemctl disable triggerhappy
```

---

## 6. Optional: reduce GPU memory

For headless operation, free up RAM by reducing GPU memory split in `/boot/firmware/config.txt`:

```
gpu_mem=16
```

---

## 7. Reboot and verify

```bash
sudo reboot
```

After reboot, verify tmpfs mounts are active:

```bash
df -h | grep tmpfs
```

You should see entries for `/tmp`, `/var/log`, `/var/spool`, `/var/tmp`.

Check that no swap is active:

```bash
swapon --show   # Should return nothing
```

---

## Summary of what writes to SD card after this setup

| Path | Location | Notes |
|------|----------|-------|
| Root filesystem | SD card | Writes only on config changes / updates |
| `/tmp` | RAM | Cleared on reboot |
| `/var/log` | RAM | Logs lost on reboot |
| `/var/spool` | RAM | Print/mail queues etc. |
| Swap | None | Disabled |
| Journal | RAM | Lost on reboot |

**Effective SD card writes reduced to: OS updates and your application writing persistent data.**

---

## Optional: persist a minimal crash log

If you want *some* log persistence for diagnostics without hammering the SD card, you can use `log2ram` as an alternative to the manual `/var/log` tmpfs above. It keeps logs in RAM but syncs to SD on shutdown/reboot.

```bash
echo "deb [signed-by=/usr/share/keyrings/azlux-archive-keyring.gpg] http://packages.azlux.fr/debian/ bookworm main" \
  | sudo tee /etc/apt/sources.list.d/azlux.list
sudo wget -O /usr/share/keyrings/azlux-archive-keyring.gpg \
  https://azlux.fr/repo.gpg
sudo apt update && sudo apt install log2ram
```

Config: `/etc/log2ram.conf` — adjust `SIZE` if your logs need more than the default 40M.

> Do **not** combine log2ram with the manual `tmpfs /var/log` fstab entry — pick one approach.

---

## Maintenance notes

- **Updates:** `sudo apt update && sudo apt full-upgrade -y` works normally — root filesystem is still writable.
- **Application data** that must survive reboot should be written to a dedicated path on the SD card (e.g. `/var/lib/yourapp`), not under `/var/log` or `/tmp`.
- **Power loss:** With this setup, a hard power cut is safe for the filesystem in the sense that the volatile directories are RAM-only anyway. The root filesystem is still susceptible to corruption on unclean shutdown — use a proper shutdown procedure where possible, or consider adding a watchdog + UPS for critical applications.
