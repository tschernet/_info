# Raspberry Pi WiFi AP Fallback
## Automatic access point when no WiFi connection is available

**Use case:** Headless Pi appliance that connects to a known WiFi network in normal operation. If no connection is established within 2 minutes of boot, it falls back to a local access point for configuration — useful for first-time deployment or on-site WiFi setup.

**Tested on:** Raspberry Pi Zero 2 W, Raspberry Pi OS Lite (Bookworm)

> ⚠️ **AP+STA mode caveat:** The Pi Zero 2 W has a single WiFi chip. Running AP and client mode simultaneously on one radio works but can be flaky. Test on the bench before deploying on-site.

---

## 1. Install RaspAP

```bash
curl -sL https://install.raspap.com | bash
```

This installs:
- A WiFi AP (default SSID: `raspi-webgui`, password: `ChangeMe`)
- Web configuration UI at `http://10.3.141.1`
- DHCP server for AP clients

After installation, **stop and disable RaspAP for now** — the fallback script will start it on demand:

```bash
sudo systemctl stop raspapd
sudo systemctl disable raspapd
```

---

## 2. Create the fallback script

```bash
sudo nano /usr/local/bin/wifi-fallback.sh
```

```bash
#!/bin/bash
set -x

# Wait up to 2 minutes for a WiFi connection
# Checks every 5 seconds, 24 iterations = 120 seconds
for i in $(seq 1 24); do
    if iwgetid wlan0 -r | grep -q .; then
        echo "WiFi connected, no fallback needed"
        exit 0
    fi
    sleep 5
done

# No connection after 2 minutes — start RaspAP
echo "No WiFi found after 2 minutes, starting AP fallback"
systemctl start raspapd
```

Make it executable:

```bash
sudo chmod +x /usr/local/bin/wifi-fallback.sh
```

---

## 3. Create the systemd service

```bash
sudo nano /etc/systemd/system/wifi-fallback.service
```

```ini
[Unit]
Description=WiFi fallback AP if no connection after boot
After=network-online.target
Wants=network-online.target

[Service]
Type=oneshot
ExecStart=/usr/local/bin/wifi-fallback.sh
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
```

Enable it:

```bash
sudo systemctl daemon-reload
sudo systemctl enable wifi-fallback.service
```

---

## 4. Usage

### Normal operation
Pi connects to known WiFi → fallback script exits cleanly → RaspAP stays off → no AP visible.

### No WiFi available (first deployment / unknown network)
After 2 minutes with no connection:
1. RaspAP starts
2. AP appears as `raspi-webgui`
3. Connect from phone or laptop (password: `ChangeMe`)
4. Open browser → `http://10.3.141.1`
5. Configure client WiFi to point at the local network
6. Reboot Pi → now connects normally

### Check fallback status
```bash
journalctl -u wifi-fallback.service
systemctl status raspapd
```

---

## 5. Post-deployment cleanup (optional)

Once the Pi is permanently connected to the target WiFi, RaspAP can be removed to keep the system lean:

```bash
# Disable the fallback service
sudo systemctl disable wifi-fallback.service

# Remove RaspAP
# Check RaspAP docs for current uninstall procedure:
# https://docs.raspap.com
```

Or leave it in place — handy if WiFi credentials change or the Pi moves to a new location.

---

## Tweaking the timeout

To adjust the wait time, change the loop parameters in the script:

| Iterations | Interval | Total wait |
|-----------|----------|------------|
| 12 | 5s | 1 minute |
| 24 | 5s | 2 minutes (default) |
| 36 | 5s | 3 minutes |

Or change the sleep interval — e.g. `sleep 10` with 12 iterations = 2 minutes with less CPU wakeups.

---

## Notes

- RaspAP default credentials should be changed for anything beyond bench use — change SSID/password via the web UI after first login.
- The fallback AP and the client WiFi share the same radio on Pi Zero 2 W — throughput and stability may be reduced while both are active simultaneously.
- This setup plays nicely with the [appliance hardening guide](rpi-appliance-hardening.md) — the fallback script and service add negligible SD card writes.
