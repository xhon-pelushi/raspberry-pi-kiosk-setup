
# 🖥️ Raspberry Pi Kiosk Setup (2023+)

This guide will walk you through configuring a Raspberry Pi to launch directly into Chromium in kiosk mode.

---

## 📥 1. Install Raspbian OS

- Download the official Raspberry Pi Imager: [https://www.raspberrypi.com/software/](https://www.raspberrypi.com/software/)
- Choose:
  - **OS**: Raspberry Pi OS (32-bit)
  - **Storage**: Your SDHC Card
- Click **"Write"** to install the OS.

---

## 🔌 2. Boot & Initial Configuration

- Insert SD card into the Pi and power it on.
- Connect to Wi-Fi.
- Open Terminal and run:

```bash
sudo apt-get update && sudo apt-get upgrade -y
```

---

## 🌐 3. Install Chromium

```bash
sudo apt-get install chromium x11-xserver-utils unclutter
sudo apt install rpi-chromium-mods
```

---

## 🔐 4. Auto-login & Networking Setup

Run:

```bash
sudo raspi-config
```

- **System Options**:
  - `S3 Password` → Set local admin password
  - `S6 Network at Boot` → Wait until network is available
  - `S5 Boot/Auto Login` → `B4: Auto-login as pi user to Desktop`
- **Interface Options**:
  - `I2 SSH` → Enable SSH

---

## 📜 5. Create Kiosk Script

```bash
nano /home/pi/kiosk.sh
```

Paste the following:

```bash
#!/bin/bash
xset s noblank
xset s off
xset -dpms

sed -i 's/"exited_cleanly":false/"exited_cleanly":true/' /home/pi/.config/chromium/Default/Preferences
sed -i 's/"exit_type":"Crashed"/"exit_type":"Normal"/' /home/pi/.config/chromium/Default/Preferences

/usr/bin/chromium-browser --noerrdialogs --disable-infobars --kiosk "https://bit.ly/"

while true; do
  sleep 10
done
```

---

## ⚙️ 6. Create Service to Auto-Start Kiosk

```bash
sudo nano /lib/systemd/system/kiosk.service
```

Paste:

```ini
[Unit]
Description=Chromium Kiosk
Wants=graphical.target
After=graphical.target

[Service]
Environment=DISPLAY=:0.0
Environment=XAUTHORITY=/home/pi/.Xauthority
Type=simple
ExecStart=/bin/bash /home/pi/kiosk.sh
Restart=on-abort
User=pi
Group=pi

[Install]
WantedBy=graphical.target
```

---

## 🛑 7. Prevent Screen Sleep

```bash
sudo nano /etc/xdg/lxsession/LXDE-pi/autostart
```

Add the following:

```bash
xset s noblank
xset s off
xset -dpms
```

---

## 🔁 8. Enable & Start Kiosk Service

```bash
sudo systemctl enable kiosk.service
sudo systemctl start kiosk.service
```

To verify the status:

```bash
sudo systemctl status kiosk.service
```

---

## ⏰ 9. Auto-Restart Pi Every 4 Hours

```bash
sudo crontab -e
```

At the end, add:

```bash
*/240 * * * * /sbin/shutdown -r now
```

---

## 🔄 10. Reboot to Test

```bash
sudo reboot
```

Your Raspberry Pi should now boot directly into Chromium in kiosk mode.

---

*Created and maintained by Xhon Pelushi. Contributions and suggestions are welcome!*