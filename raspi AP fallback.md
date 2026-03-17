# Raspberry Pi WiFi AP Fallback (nmcli variant)
## Lightweight automatic access point when no WiFi connection is available

**Use case:** Headless Pi appliance that connects to a known WiFi network in normal operation. If no connection is established within 2 minutes of boot, it falls back to an open access point for SSH access and manual configuration.

**Tested on:** Raspberry Pi Zero 2 W, Raspberry Pi OS Lite (Bookworm)

**Advantage over RaspAP:** No extra software needed — uses NetworkManager which is already present on Raspberry Pi OS.

> ⚠️ **AP+STA mode caveat:** The Pi Zero 2 W has a single WiFi chip. Running AP and client mode simultaneously on one radio works but can be flaky. Test on the bench before deploying on-site.

> ⚠️ **Open AP:** No password — intended for temporary configuration access only, not permanent operation.

---

## Prerequisites

Verify NetworkManager is running and managing wlan0:

```bash
systemctl status NetworkManager
nmcli device status
```

`wlan0` should appear in the nmcli output. If NetworkManager is not present:

```bash
sudo apt install network-manager
```

---

## 1. Create the fallback script

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

# No connection after 2 minutes — start open config AP
echo "No WiFi found after 2 minutes, starting config AP"
nmcli device wifi hotspot ifname wlan0 ssid kumm-config password ""
```

Make it executable:

```bash
sudo chmod +x /usr/local/bin/wifi-fallback.sh
```

---

## 2. Create the systemd service

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

## 3. Usage

### Normal operation
Pi connects to known WiFi → fallback script exits cleanly → no AP visible.

### No WiFi available (first deployment / unknown network)
After 2 minutes with no connection:
1. Open AP appears as `kumm-config` (no password)
2. Connect from phone or laptop
3. SSH in: `ssh kumm@10.42.0.1`
4. Edit `/etc/wpa_supplicant/wpa_supplicant.conf` or use `nmcli` to add the new network:
```bash
sudo nmcli device wifi connect "SSID" password "wifipassword"
```
5. Reboot → Pi connects normally

### Manually trigger the AP (without rebooting)
```bash
sudo nmcli device wifi hotspot ifname wlan0 ssid kumm-config password ""
```

### Turn off the AP manually
```bash
sudo nmcli connection down Hotspot
```

### Check fallback status
```bash
journalctl -u wifi-fallback.service
```

---

## Tweaking the timeout

To adjust the wait time, change the loop parameters in the script:

| Iterations | Interval | Total wait |
|-----------|----------|------------|
| 12 | 5s | 1 minute |
| 24 | 5s | 2 minutes (default) |
| 36 | 5s | 3 minutes |

---

## 4. Commandline Wifi Management with NetworkManager: nmcli

scan networks in range: 
```bash
sudo nmcli device wifi list
```

connect to new wifi:
```bash
sudo nmcli device wifi connect "SSID" password "wifipassword"
```

list configured networks: 
```bash
nmcli connection show
```

add wifi without connecting: 
```bash
sudo nmcli connection add type wifi ssid "SSID" -- wifi-sec.key-mgmt wpa-psk wifi-sec.psk "wifipassword"
```

## Notes

- SSH gateway IP when connected to the AP: `10.42.0.1`
- SSID can be changed to anything — edit the script and the manual trigger command accordingly
- This setup plays nicely with the [appliance hardening guide](rpi-appliance-hardening.md)
- No extra packages required — NetworkManager is already present on Raspberry Pi OS Lite (Bookworm)
