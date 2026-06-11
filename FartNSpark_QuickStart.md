# Fart 'N Spark — Quick Start Guide

> ⚠️ **Safety first.** This device switches mains-voltage (230 V AC) equipment. Only connect it to inverters that are already safely installed by a qualified electrician. Never touch wiring while the system is live.

---

## What you need

- Wemos D1 Mini (ESP8266, 4 MB flash)
- USB-A to Micro-USB cable
- One or more Shelly Plug S (Gen 1, 2, or 3)
- WattWächter energy meter (publishing MQTT)
- A Windows, Mac, or Linux computer with Chrome or Edge browser
- Your home Wi-Fi name and password
- Your MQTT broker address and credentials

---

## Step 1 — Flash the firmware

1. Download the latest `.bin` file from the [Releases](../../releases) page.
2. Connect the Wemos D1 Mini to your computer via USB.
3. Open **Chrome** or **Edge** and go to **https://esp.huhn.me**
4. Click **Connect** and select the Wemos COM port from the list.
5. Click **Add file**, set the address to `0x0`, and select the `.bin` file.
6. Click **Program** and wait ~30 seconds until it says Done.
7. Unplug and replug the Wemos.

> Firefox does not work — use Chrome or Edge only.

---

## Step 2 — Connect to the device

1. **On your phone or laptop, turn off mobile data (LTE/4G/5G).** This is important — your device will route traffic via mobile data instead of the AP if mobile data is on.
2. Connect to the Wi-Fi network **FartNSpark** with password **deye1234**.
3. Open a browser and go to **http://192.168.4.1**
4. You should see the Fart 'N Spark web interface.

---

## Step 3 — Configure Wi-Fi and MQTT

1. Click the **Configuration** tab.
2. Under **Wi-Fi**, enter your home network name and password.
3. Under **WattWächter MQTT**, enter:
   - **MQTT Host** — IP address of your MQTT broker (e.g. `192.168.1.10`)
   - **MQTT Port** — usually `1883`
   - **MQTT User / Password** — if your broker requires login
   - **MQTT Topic** — the topic your WattWächter publishes to (e.g. `tele000999/WattWaechter/SENSOR`)
   - **Power Key** — the JSON key for grid power (e.g. `Power`)
4. Click **Save & Apply**. The device reboots and joins your Wi-Fi.
5. Re-enable mobile data and reconnect to your normal Wi-Fi.

---

## Step 4 — Add your Shelly plugs

1. Make sure each Shelly plug is on the same Wi-Fi network as the Wemos.
2. Find the IP address of each Shelly (check your router's device list or the Shelly app).
3. In the Configuration tab, scroll down to **Plugs**.
4. For each inverter:
   - Enter the Shelly's IP address
   - Give it a name (e.g. `Deye South`)
   - Set the generation (Gen 1 = original Shelly Plug S, Gen 2 = Plus Plug S, Gen 3 = newer models)
   - Enter the inverter's nominal output power in watts (e.g. `800`)
   - Tick **Enabled**
5. Click **Save & Apply**.

---

## Step 5 — Verify it works

1. Go to the **Status** tab.
2. Check that:
   - **MQTT** shows **CONNECTED**
   - **Grid Power** shows a live reading from your meter
   - Each plug card shows **Online** with a green indicator
3. Watch the log (Log tab) for control decisions — you should see lines like `Best-fit ON S1` or `Best-fit OFF S1` as the system switches inverters.

---

## Key settings to know

| Setting | Default | What it does |
|---------|---------|-------------|
| Export Threshold | 800 W | Switches inverters OFF when exporting more than this |
| ON Threshold | 0 W | Only switches inverters ON when importing (grid < 0 W) |
| Avg Window | 180 s | How many seconds of history to average. Higher = slower but more stable |
| Post-ON Lockout | 90 s | Minimum time between switching two inverters ON |
| Post-OFF Lockout | 90 s | Minimum time between switching two inverters OFF |

---

## Troubleshooting

**Can't see the FartNSpark Wi-Fi network:**
Make sure the Wemos is powered and the flash completed successfully.

**Browser shows "site can't be reached" at 192.168.4.1:**
Make sure mobile data is turned off on your phone before connecting to FartNSpark.

**MQTT shows DISCONNECTED:**
Check the broker IP, port, and credentials. Make sure the Wemos joined your Wi-Fi first (the MQTT connection only works on your home network, not in AP mode).

**Plug shows Offline:**
Check the Shelly's IP address is correct and that it's on the same network as the Wemos.

**⚠ FAULT badge on a plug:**
The plug didn't respond to a switch command within 60 seconds. Press Force ON or Force OFF to clear it. Check the Shelly is reachable.

---

## Change the AP password

The default AP password `deye1234` is the same on every device. Change it in the Configuration tab under **AP Mode Password** (minimum 8 characters). The current password is always shown in the Serial Monitor at 115200 baud on boot.

---

*For full documentation see [FartNSpark_Manual_v4011.md](FartNSpark_Manual_v4011.md)*
