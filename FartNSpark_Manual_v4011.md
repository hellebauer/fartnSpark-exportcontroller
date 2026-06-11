# Fart 'N Spark — User Manual

**Solar Export Controller · Shelly Edition · Firmware v4.011**

---

## Contents

1. [Safety & Legal Notice](#1-safety--legal-notice)
   - Disclaimer of liability
   - Electrical hazards
   - Fire hazard
   - Grid and regulatory hazards
   - Cybersecurity hazards
   - Software reliability
   - Qualified installation requirement
2. [How It Works](#2-how-it-works)
3. [Hardware & Wiring](#3-hardware--wiring)
   - Inverter power vs Shelly rated load
4. [Flashing the Firmware](#4-flashing-the-firmware)
5. [First-Boot Setup](#5-first-boot-setup)
6. [Web Interface](#6-web-interface)
7. [Configuration Reference](#7-configuration-reference)
8. [Control Algorithm](#8-control-algorithm)
9. [Relay Cycle Counter & Lifespan](#9-relay-cycle-counter--lifespan)
   - Effect of inverter power on relay lifespan
10. [API Reference](#10-api-reference)
11. [Troubleshooting](#11-troubleshooting)
12. [MQTT & WattWächter Integration](#12-mqtt--wattwächter-integration)
13. [Appendix A — Factory Reset](#13-appendix-a--factory-reset)
14. [Appendix B — Firmware Version History](#14-appendix-b--firmware-version-history)
   - v4.004–v4.009 changes

---

## 1  Safety & Legal Notice

> ⚠️ **READ THIS SECTION IN FULL BEFORE INSTALLING OR USING THIS FIRMWARE.**
> Failure to observe these warnings may result in **electric shock, fire, serious injury, or death**, and may cause damage to property, connected equipment, or the public electricity grid.
> **If you do not understand any part of this section, do not proceed. Consult a qualified electrician.**

---

### 1.1  Disclaimer of liability

**THIS FIRMWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE, AND NON-INFRINGEMENT.**

The author(s) of Fart 'N Spark accept **no liability whatsoever** for any of the following, whether arising from use, misuse, correct use, installation, configuration, or any failure of the firmware or associated hardware:

- Death or personal injury
- Electric shock or electrocution
- Fire, explosion, or thermal damage
- Damage to property, buildings, or infrastructure
- Damage to PV inverters, Shelly plugs, the Wemos D1 Mini, or any connected equipment
- Damage to the public electricity grid or grid-connected equipment belonging to third parties
- Financial loss, including loss of feed-in tariff income, energy costs, or consequential losses
- Data loss or corruption
- Regulatory fines, penalties, or legal costs arising from non-compliant grid feed-in
- Any indirect, incidental, special, exemplary, or consequential damages of any kind

**By installing or using this firmware, you accept sole and full responsibility for all consequences of its use.** If you do not accept these terms, do not install or use the firmware.

---

### 1.2  Electrical hazards — DANGER

> 🔴 **DANGER: MAINS VOLTAGE.** The Shelly plugs controlled by this firmware carry **230 V AC** (or the mains voltage of your region). Contact with live mains voltage is likely to cause **severe injury or death**.

- **Only qualified electricians** may install, wire, or service mains-voltage equipment. This includes the connection of Shelly plugs to PV inverter AC outputs.
- Never open, modify, or probe a Shelly plug while it is connected to mains power, even if the relay is in the OFF state — the input terminals remain live at all times.
- Never touch internal components of a Shelly plug without verifying that all upstream circuit breakers are open and locked out.
- The Wemos D1 Mini operates at **3.3 V DC**. It must never be connected directly or indirectly to mains voltage. Power it only from a suitable 5 V USB supply.
- Do not use damaged Shelly plugs or cables. Inspect equipment for signs of burning, melting, or physical damage before energising.
- Ensure all Shelly plugs are rated for the full output current of the connected PV inverter, including surge currents at switch-on.
- Do not exceed the rated load of any Shelly plug. Overloading a relay contact will cause overheating and may cause fire.
- Ensure equipment is installed in a location appropriate for the IP rating of the devices. Do not install in wet, damp, or dusty environments unless the devices are rated for it.

---

### 1.3  Fire hazard

- Relay contacts switching inductive or capacitive loads (including inverter output circuits) may produce arcing. Use only plugs and switchgear rated for the load type.
- Do not leave equipment unattended during initial commissioning.
- Ensure adequate ventilation around all equipment. Do not cover or insulate Shelly plugs during operation.
- An incorrectly configured firmware (e.g. rapid cycling due to wrong `avgWindowSec` or `offLockoutSec` settings) may cause relay contacts to overheat through excessive switching. Monitor relay cycle counts and lifespan estimates shown in the web UI.

---

### 1.4  Grid and regulatory hazards

- **Grid feed-in is regulated in most jurisdictions.** Connecting PV inverters to the public grid without the knowledge and approval of your electricity distribution network operator (DNO / grid operator) may be illegal and may void your energy supply contract, insurance, and building warranty.
- This firmware does **not** implement:
  - Over-frequency ride-through or tripping
  - Anti-islanding protection (this is the sole responsibility of the inverters themselves)
  - Voltage rise mitigation
  - Any grid protection function required by standards such as VDE-AR-N 4105, G98, G99, EN 50549, or equivalent regional standards
- The firmware is a **switching automation tool only**. Compliance with all applicable grid connection standards, energy regulations, and building codes is entirely the responsibility of the installer and system owner.
- Check with your DNO, energy provider, and local authority before commissioning any grid-connected PV system or modifying an existing one.

---

### 1.5  Cybersecurity hazards

- The web interface runs on port 80 with **no authentication**. Do not expose the device directly to the internet. Place it behind a firewall or on an isolated IoT VLAN.
- The default AP password (`deye1234`) and MQTT broker (`broker.hivemq.com`) are publicly known. Change them before deployment in any production environment.
- MQTT traffic is transmitted in plaintext. Do not transmit sensitive credentials over an untrusted network.
- An attacker with network access to the device can switch inverters on or off at will. Assess the physical and electrical consequences of unauthorised switching before deployment.

---

### 1.6  Software reliability

- This firmware is **hobbyist / experimental software**. It has not been certified, audited, or approved by any regulatory body.
- The firmware may contain bugs that cause incorrect switching behaviour, including switching all inverters off (loss of self-consumption) or failing to switch them off (uncontrolled export).
- The moving average and control algorithm are not guaranteed to keep grid export below any legal or contractual limit under all conditions.
- The device may reboot unexpectedly due to watchdog resets, heap exhaustion, or Wi-Fi disconnection. During a reboot, all plugs retain their last physical state — there is no guaranteed fail-safe position at power-up.
- **Do not rely on this firmware as the sole protection against over-export.** A certified energy management system or a hardware export limiter should be used where export limits are legally binding.

---

### 1.7  Qualified installation requirement

Installation of this system involves work on or near mains-voltage equipment. In most jurisdictions this work:

- Must be carried out by a licensed or registered electrician
- Must comply with local wiring regulations (e.g. BS 7671, DIN VDE 0100, NEC, or equivalent)
- May require notification to or approval from the DNO or grid operator
- May require a building permit or electrical installation certificate

The firmware author(s) accept no responsibility for installations that do not comply with applicable regulations.

---

### 1.8  Summary of user responsibilities

By using this firmware, you confirm that you:

1. Have read and understood this entire safety and legal notice.
2. Are a qualified electrician, or are working under the direct supervision of one.
3. Have obtained all necessary permits and approvals from your DNO, energy provider, and local authority.
4. Accept full legal and financial responsibility for the installation, operation, and consequences of this firmware.
5. Will not hold the firmware author(s) liable for any damages, losses, or legal consequences of any kind.

---

## 2  How It Works

Fart 'N Spark reads the current grid exchange power (positive = export, negative = import) from a WattWächter energy meter via MQTT. It compares a smoothed average of this reading against a configurable export threshold (default 800 W) and switches PV inverters on or off as needed.

### 2.1  Physics of switching

- **Switching OFF is instantaneous.** The Shelly relay opens and the inverter stops feeding in immediately. If the inverter was delivering 600 W, exactly 600 W disappears from the grid — no ramp, no delay.
- **Switching ON takes time.** The inverter must re-synchronise to the grid (per VDE / EN 50549) before it resumes output. This ramp-up can take from a few seconds to several minutes depending on the model. For Deye M80G4 units, measure this on your installation — it can exceed 90 seconds.
- **Shelly measures what passes through it.** A plug reporting 0 W is genuinely delivering 0 W. Zero-watt readings during ramp-up are real and expected.

### 2.2  Moving average window

Instead of reacting to the raw instantaneous reading, the controller maintains a moving average over a configurable window (default 180 s). This has two major benefits:

- **Stove / kitchen appliance immunity.** A 2 kW hob cycling on and off every 15 s produces a 300 W average swing — invisible through a 180 s window. Without averaging, the same load would trigger 40+ relay operations per day.
- **Cloud shadow immunity.** Passing clouds cause rapid export swings. The moving average absorbs these without unnecessary switching.

Simulated impact of window length on a Berlin installation:

| Window | Relay ops / year | S1 lifetime (est.) | Cloud immunity |
|--------|-----------------|-------------------|----------------|
| 60 s | ~75,000 | ~3 years | Moderate |
| 180 s (default) | ~25,000 | ~10 years | Good |
| 300 s | ~12,000 | >20 years | Excellent |

### 2.3  Energy budget (15-minute slot)

In addition to the instantaneous threshold, an energy budget limits total export per 15-minute slot (configurable). The default budget of 200 Wh at the default 800 W threshold acts as a background safety net — it only becomes binding if you deliberately raise the instantaneous threshold above 800 W to allow short export bursts while keeping average export controlled.

> **Note:** With default settings (threshold 800 W, budget 200 Wh, window 15 min), the budget constraint and the instantaneous threshold are numerically equivalent. The budget only matters if you raise the instantaneous threshold.

### 2.4  Best-fit plug selection

When an action is needed, the controller picks the single most appropriate plug rather than switching all available plugs at once:

- **Excess (OFF branch):** Find the plug whose power is closest to the excess from above. If no single plug covers the excess, choose the largest.
- **Headroom (ON branch):** Find the plug whose nominal power is closest to the available headroom from below.
- **Keep-strongest rule:** When deciding which plug to switch off, the highest-power plug that alone keeps export under the threshold is protected from switching. This prevents oscillation caused by ramping inverters.

---

## 3  Hardware & Wiring

### 3.1  Bill of materials

| Item | Qty | Notes |
|------|-----|-------|
| Wemos D1 Mini (ESP8266) | 1 | 4 MB flash version |
| Micro-USB power supply | 1 | 5 V, ≥ 500 mA |
| Shelly Plug S / Plus Plug S | 1–8 | Gen 1, 2, or 3 all supported |
| WattWächter (Tasmota meter) | 1 | Must publish MQTT grid power |
| 2.4 GHz Wi-Fi access point | 1 | Same LAN as Shellys + WattWächter |
| MQTT broker | 1 | broker.hivemq.com (public) or local Mosquitto |

### 3.2  Wiring overview

No custom wiring between the Wemos and the Shellys is required. All communication is over your existing Wi-Fi network.

```
PV inverter 1  →  AC output  →  Shelly Plug 1  ─── Wi-Fi ───┐
PV inverter 2  →  AC output  →  Shelly Plug 2  ─── Wi-Fi ───┤
   …                               …                         │── Wi-Fi AP ── Wemos D1 Mini
PV inverter N  →  AC output  →  Shelly Plug N  ─── Wi-Fi ───┘

Grid / Meter   ←→ current →    WattWächter     ─── MQTT  ───┘  (→ broker)
```

> **Tip:** The Wemos D1 Mini only needs USB power and Wi-Fi. No GPIO connections to the Shellys are required.

### 3.3  Shelly generation identification

| Generation | Model examples | API used | Web UI colour |
|-----------|----------------|----------|--------------|
| Gen 1 | Shelly Plug S (original) | `/status` (JSON nested) | Green |
| Gen 2 | Shelly Plus Plug S | `/rpc/Switch.GetStatus?id=0` (flat JSON) | Dark |
| Gen 3 | Shelly Plus Plug IT etc. | Treated as Gen 2 | Dark |

### 3.4  Inverter power vs Shelly rated load

> ⚠️ **The nominal output power of each PV inverter must not exceed the rated load of the Shelly plug it is connected to.**

| Shelly model | Max continuous load |
|-------------|-------------------|
| Plug S Gen 1 | 2300 W / 10 A |
| Plus Plug S Gen 2 | 2500 W / 10 A |
| Plus Plug IT Gen 3 | 2500 W / 10 A |

A Shelly relay switching a load close to its rated current will run hot, arc heavily at contact break, and wear out significantly faster than one switching a lighter load. Where possible, pair each Shelly with an inverter well below the plug's rated capacity — for example, an 800 W microinverter on a 2500 W plug runs the relay at 32% of rated current, which is far gentler than a 2300 W inverter at 100%.

Surge currents at inverter switch-on can briefly exceed the steady-state output. Ensure the Shelly is rated for the peak inrush current of your inverter model, not just the nominal output.

---

## 4  Flashing the Firmware

The firmware is flashed once via USB using the Arduino IDE. There is no OTA update mechanism — subsequent updates also require USB.

### 4.1  Arduino IDE settings

| Setting | Value |
|---------|-------|
| Board | LOLIN(WEMOS) D1 mini (clone) |
| Flash size | 4MB (FS: 1MB LittleFS) |
| Upload speed | 921600 |
| CPU frequency | 80 MHz |

### 4.2  Required libraries

> ⚠️ **Critical:** ArduinoJson MUST be the first `#include` in the sketch. ESPAsyncWebServer bundles an older version that shadows the newer one if included first, causing `JsonDocument not declared` compile errors.

| Library | Author | Notes |
|---------|--------|-------|
| ArduinoJson v7 | Benoit Blanchon | Must be `#include`'d first |
| ESPAsyncTCP | dvarrel | |
| ESPAsyncWebServer | Hristo Gochkov | |
| PubSubClient | Nick O'Leary | MQTT client |
| LittleFS | ESP8266 core | Built-in, no install needed |

### 4.3  Flash procedure

1. Install the Arduino IDE and the ESP8266 board package (`https://arduino.esp8266.com/stable/package_esp8266com_index.json`).
2. Install all five libraries via the Library Manager.
3. Open `deye_shelly_8266V1_v4003.ino` in the Arduino IDE.
4. Connect the Wemos D1 Mini via micro-USB.
5. Select the correct COM / tty port.
6. Select **Sketch → Upload**. The upload takes about 10 seconds.
7. Open the Serial Monitor at 115200 baud to watch the boot log.

---

## 5  First-Boot Setup

On first boot with no saved configuration, the device starts as a Wi-Fi access point (AP) so you can enter your network credentials.

### 5.1  Connect via access point

1. **Disable mobile data (LTE/4G/5G) on your phone before connecting.** iOS and Android silently route traffic via mobile data when a Wi-Fi network has no internet connection — the AP has no internet, so all browser requests will be routed away from the device. The config page will appear unreachable until mobile data is disabled.
2. Join the Wi-Fi network **FartNSpark**. The AP password is printed in the Serial boot log — check the Serial Monitor at 115200 baud. The factory default is `deye1234`. Change it in the Configuration tab once connected.
3. Open a browser and navigate to `http://192.168.4.1` (not https://).
4. Go to the **Configuration** tab.
5. Enter your home Wi-Fi SSID and password under the **Wi-Fi** section.
6. Click **Save & Apply**. The device will reboot and join your network.

> **Note:** The access point is only active when the device cannot join a configured Wi-Fi network. Once connected, the AP is completely off.

### 5.2  Find the device on your network

After the reboot, find the device IP from your router's DHCP list, or check the Arduino Serial Monitor — the boot log prints the assigned IP address. Then open `http://<device-ip>` in any browser.

### 5.3  Minimum configuration checklist

Complete these steps before the system will control any plugs:

- **Wi-Fi:** SSID and password (done in step 5.1).
- **MQTT broker:** Host, port (default 1883), optional user/password, topic, and power key. See [section 12](#12-mqtt--wattwächter-integration) for WattWächter integration details.
- **Shelly plugs:** IP address, name, generation, and nominal power (`nomW`) for each plug. Enable the plug with the checkbox.
- **Export threshold:** Set to your desired maximum grid export in watts (default 800 W).
- **nomW (strongly recommended):** Enter each inverter's rated power. Without `nomW`, the keep-strongest rule and best-fit selection degrade significantly during inverter ramp-up.

### 5.4  Verify operation

1. Check the **Status** tab — the MQTT badge should show **CONNECTED**.
2. Verify grid power is updating. The **Grid Power** card should show a live reading.
3. Check that your Shelly plugs show **Online** with a green dot.
4. Watch the **Log** tab — you should see control decisions appearing every 5 seconds (e.g. `Best-fit OFF S2 (200.0W excess=150.0W)`).

---

## 6  Web Interface

The web UI is a single-page application served directly from the device. The refresh interval automatically matches the control interval (default 5 s, configurable). Three tabs organise the content.

### 6.1  Header

The header shows the product name and, on the right, live system stats:

| Field | Description |
|-------|-------------|
| Uptime | Time since last reboot |
| Free Heap | Available RAM. Below ~8 kB the system auto-reboots |
| IP | Device IP address on your Wi-Fi network |
| RSSI | Wi-Fi signal strength in dBm |
| Net | Connected Wi-Fi SSID |

### 6.2  Status tab

#### Summary cards

| Card | Description |
|------|-------------|
| Grid Power | Instantaneous grid reading from MQTT (+ = export, − = import) |
| Fleet Status | ON/OFF badge showing whether any inverter is currently active |
| MQTT / WattWächter | CONNECTED (green) or DISCONNECTED (red); turns stale after 60 s without a new reading |

#### Plug status cards

Each configured plug shows:

- Plug number and Shelly generation badge
- Plug name and IP address
- Online / Offline indicator (green / red dot)
- ON/OFF state badge and current power reading in watts
- **Cycle counter badge** (top-right corner) — bordered box showing relay On/Off cycle count since last reboot and estimated remaining lifespan (years). Each ON or OFF counts as 0.5 cycles. Resets to zero on every reboot.
- **⚠ FAULT badge** — shown in red next to the ON/OFF state badge when the plug has not reached its commanded state within `faultTimeoutSec` seconds. Clears automatically when the plug responds. Use Force ON/OFF to reset manually.
- **Force ON / Force OFF** buttons — bypass the control algorithm for manual testing. Force commands bypass rate limiting.

> ⚠️ **Warning:** Force ON/OFF bypasses rate limiting by resetting `lastToggleMs` to zero. The control algorithm resumes normal operation on the next control cycle (within 5 seconds).

### 6.3  Log tab

Shows the last 1200 characters of the on-device log buffer. Timestamps are shown as seconds-since-boot in `[Xs]` format; the browser converts these to real wall-clock times using the current date and device uptime.

The **Heap Monitor** at the bottom shows a Chart.js sparkline of free heap memory sampled hourly over the last 7 days. Each hourly data point represents the **minimum** free heap observed during that hour, so the chart shows worst-case conditions rather than idle snapshots. A steady decline may indicate a memory leak; a sudden drop followed by recovery indicates an auto-reboot.

### 6.4  Configuration tab

All settings are edited here and saved to the device's internal flash (LittleFS) by clicking **Save & Restart**. The device reboots 400 ms after saving.

Buttons in the Configuration tab:

| Button | Action |
|--------|--------|
| **Save & Apply** | Saves all settings to flash and reboots after 400 ms. |
| **Discard Changes** | Reloads the current saved configuration, discarding any unsaved edits in the form. |
| **Reboot** | Reboots the device without changing any settings. Useful after firmware flash. |
| **⚠ Factory Reset** | Deletes `config.json` and reboots into AP mode. Requires confirmation. |

> ⚠️ **Warning:** After flashing new firmware, always do a hard refresh (Ctrl+Shift+R / Cmd+Shift+R) in your browser. The device serves the HTML from a LittleFS cache keyed to the firmware version number — a plain refresh may serve the old page.

---

## 7  Configuration Reference

### 7.1  Wi-Fi

| Parameter | Description |
|-----------|-------------|
| Wi-Fi SSID | Your 2.4 GHz network name. 5 GHz not supported. |
| Wi-Fi Password | Network password. Stored in flash, not returned by API. |
| AP Mode Password | Password for the fallback AP (min 8 characters). Default: `deye1234`. A red warning is shown while the default is still in use. The current password is always printed in the Serial boot log. |
| Timezone | POSIX timezone string. Select from the 46-option dropdown. Default: Germany CET/CEST (`CET-1CEST,M3.5.0,M10.5.0/3`). Used for log timestamps only. |

### 7.2  MQTT

| Parameter | Description |
|-----------|-------------|
| Host | MQTT broker hostname or IP. Leave blank to disable MQTT entirely. |
| Port | Default: 1883. |
| User / Password | Optional broker credentials. |
| Topic | Full MQTT topic to subscribe to. Default: `tele/WattWaechter/SENSOR` |
| Power Key | JSON key inside the payload that contains grid power in watts. Default: `grid_power`. See [section 12](#12-mqtt--wattwächter-integration) for fallback key logic. |

> **Note:** MQTT connection is skipped entirely if the Host field is empty. Without MQTT the controller has no grid power data and will not switch any plugs.

### 7.3  Control thresholds

| Parameter | Default | Description |
|-----------|---------|-------------|
| Export Threshold (W) | 800 | Maximum allowed grid export. When the smoothed average exceeds this, an inverter is switched off. |
| Control & Screen Update Interval (s) | 5 | How often `runControl()` fires and how often the web UI refreshes. Range: 5–60 s. |
| Min time between ON commands (per plug, in seconds) | 120 | Minimum time between two consecutive ON commands to the same plug. Prevents rapid re-energising during ramp-up. |
| Min time between OFF commands (per plug, in seconds) | 10 | Minimum time between two consecutive OFF commands to the same plug. |
| Avg Window (s) | 180 | Moving average window length. Range: 5–300 s. Longer = more stable, slower to react. |
| Slot Budget (Wh) | 200 | Maximum export energy per time slot. Only binding when the instantaneous threshold is set above the equivalent average. |
| Slot Length (min) | 15 | Duration of one energy budget slot. |
| ON Threshold (W) | 0 | The ON branch is only active when `smoothPower` is below this value. Default 0 W means the controller only switches inverters ON when importing from the grid. Set to e.g. 400 W to allow switching ON while exporting up to 400 W. Set to a negative value to require deeper import before switching ON. |
| Relay state error confirmed after (seconds) | 60 | If a plug does not reach its commanded state within this time, it is flagged FAULT and excluded from control decisions. Set to 0 to disable. |
| Post-OFF Lockout (s) — across all plugs | 90 | After any plug is switched off, no further OFF commands are issued for this many seconds (across all plugs). Prevents overshoot from moving average lag. Set to 0 to disable. |
| Post-ON Lockout (s) — across all plugs | 90 | After any plug is switched on, no further ON commands are issued for this many seconds (across all plugs). Prevents back-to-back switching of multiple inverters before the first has ramped up. Set to 0 to disable. |
| Relay Rated Cycles | 100,000 | Manufacturer-rated relay lifetime used for the lifespan estimate on plug cards. Check your Shelly datasheet. |
| MQTT Failsafe | S1 only | What to do when MQTT data is stale for >60 s. Options: All OFF, All ON, S1 only. |

### 7.4  Shelly plug configuration

| Parameter | Description |
|-----------|-------------|
| IP Address | IPv4 address of the Shelly (e.g. `192.168.1.21`). Assign static DHCP leases from your router for reliability. |
| Name | Friendly label shown in the web UI and log messages (max 23 chars). |
| Gen | Shelly generation: 1 (original Plug S), 2 (Plus Plug S), or 3 (treated as Gen 2). |
| nomW | Inverter nominal rated power in watts. **Strongly recommended.** Used by best-fit selection and the keep-strongest rule. Without `nomW`, plug selection during ramp-up is unreliable. |
| Enabled | Must be ticked for the plug to be included in control decisions. |

---

## 8  Control Algorithm

`runControl()` fires every `ctrlIntervalSec` (default 5 s). It is skipped until the first MQTT message is received after boot (prevents switching all plugs on during startup).

### 8.1  Decision flow

1. **MQTT stale check** — If the broker is configured and no message has arrived for 60 s, apply the failsafe action and return.
2. **Compute `smoothPower`** — Calculate the moving average of the last N readings (window = `avgWindowSec`). Fall back to the instantaneous reading if fewer than 2 samples exist.
3. **Compute `dynamicThreshold`** — `min(exportThreshold, budget_remaining_W)`. With default settings these are always equal.
4. **Excess branch** (`smoothPower > dynamicThreshold`) — Check post-OFF lockout. If locked, log and return. Otherwise select best-fit plug to switch OFF (see 8.2).
5. **Headroom branch** (`smoothPower < onThreshold`) — Active only when `smoothPower` is below the configured ON threshold (default 0 W = importing). Check post-ON lockout. If locked, log and return. Otherwise select best-fit plug to switch ON (see 8.3).
6. **No action** — `smoothPower` is within the threshold band.

### 8.2  OFF branch — excess handling

`excess = smoothPower − dynamicThreshold`

1. Collect all enabled, online, ON plugs that are not rate-limited and **not faulted**.
2. **Keep-strongest rule:** Among plugs where `nomW > 0` and `nomW ≤ dynamicThreshold`, identify the one with the highest current measured power. Mark it as protected — do not switch it off this cycle.
3. From the remaining candidates, pick the plug whose power is closest to `excess` from above. If none qualifies, pick the highest-power plug.
4. Issue `shellySet(idx, false)`.
5. Set `lastOffMs = millis()` to start the post-OFF lockout timer.

> **Note:** Only one plug is switched per control cycle.

### 8.3  ON branch — headroom handling

`headroom = dynamicThreshold − smoothPower`

1. Collect all enabled, online, OFF plugs that are not rate-limited and **not faulted**.
2. Pick the plug whose `nomW` (or measured power) is closest to `headroom` from below. If none is below headroom, pick the smallest.
3. Issue `shellySet(idx, true)`.
4. Set `lastOnMs = millis()` to start the post-ON lockout timer.

### 8.4  MQTT failsafe

If no MQTT message has arrived for more than 60 seconds:

| Mode | Behaviour |
|------|-----------|
| All OFF (mode 0) | All enabled plugs are switched off. Safest — guarantees no export. |
| All ON (mode 1) | All enabled plugs are switched on. Maximises self-consumption. |
| S1 only (mode 2, default) | Plug 1 stays ON; all others are switched off. |

### 8.5  Post-OFF lockout

After any OFF action, the OFF branch is skipped until `offLockoutSec` seconds have elapsed. This prevents oscillation:

1. Excess detected → inverter X switched off
2. Moving average is slow to fall (lag) → excess still appears
3. Without lockout, another inverter is switched off unnecessarily → system undershoots to grid import

> ⚠️ **Warning:** Set `offLockoutSec` ≥ your inverter's ramp-up time. If your inverters take 90 s to reach full output after being switched on, the default 90 s lockout is appropriate. Measure this on your installation.

### 8.6  Post-ON lockout

After any ON action, the ON branch is skipped until `onLockoutSec` seconds have elapsed (default 90 s). This prevents cascading back-to-back inverter switching:

1. Grid imports detected → inverter 1 switched on
2. Moving average is slow to rise (inverter ramp-up takes 60–120 s)
3. Without post-ON lockout, inverter 2 would be switched on immediately on the next control cycle
4. Both inverters ramp up together → system overshoots to high export → both are switched off again

> ⚠️ **Warning:** Set `onLockoutSec` ≥ your inverter's ramp-up time. Measure the ramp-up duration on your installation.

### 8.7  Fault detection (Soll/Ist)

After each `shellySet()` call, the controller records the desired state and the time of the command. If the confirmed state still differs from the desired state after `faultTimeoutSec` seconds (default 60 s), the plug is flagged as FAULT. Faulted plugs are:

- Shown with a red **⚠ FAULT** badge in the web UI
- Excluded from all control decisions (ON and OFF branches)

The fault clears automatically when the plug reaches the desired state (e.g. after a network recovery). Use Force ON / Force OFF to reset a faulted plug manually.

---

## 9  Relay Cycle Counter & Lifespan

Every Shelly plug has a mechanical relay with a finite rated cycle life. Fart 'N Spark tracks relay operations and displays a real-time lifespan estimate on each plug card.

### 9.1  How cycles are counted

- Each successful relay operation (ON or OFF) counts as **0.5 cycles**. A full ON+OFF pair equals 1 cycle. Only successful HTTP 200 responses are counted — failed commands do not count.
- The counter is stored in RAM only. It **resets to zero on every reboot**.
- Force ON / Force OFF button clicks also increment the counter.

### 9.2  Lifespan calculation

```
rate            = cycles / uptime_hours
estimated_life  = rated_cycles / rate / 8760   (years)
```

Displayed as `~X.XX yr` on the plug card. Shows `---` until at least one cycle has occurred.

> **Note:** The estimate is based on the switching rate since the last reboot. During initial commissioning when many Force ON/OFF test cycles are performed, the estimate will look pessimistic — this normalises after the system has been running in production for a while.

### 9.3  Rated cycles setting

Configurable under **Configuration → Control Thresholds → Relay Rated Cycles**. Default: 100,000.

| Shelly model | Typical relay rating | Ops/year at 180 s window | Est. life |
|-------------|---------------------|-------------------------|-----------|
| Plug S Gen 1 | 50,000–100,000 | ~25,000 | 2–4 years |
| Plus Plug S Gen 2 | 100,000 | ~25,000 | ~4 years |
| Plus Plug IT Gen 3 | 100,000 | ~25,000 | ~4 years |

### 9.4  Effect of inverter power on relay lifespan

Relay wear is caused primarily by **contact arcing at the moment of switching**. Arcing energy is proportional to the current being interrupted — a relay switching 800 W (≈3.5 A) suffers significantly more arc erosion per operation than one switching 400 W (≈1.7 A).

Practical implications:

- **Lower-wattage inverters extend relay life disproportionately.** Halving the switched current reduces arcing energy by roughly 75%, potentially doubling or tripling relay lifespan.
- **The lifespan estimate in the web UI assumes constant load.** If you replace an 800 W inverter with a 400 W model, the real lifespan will be considerably better than the estimate suggests.
- **Matched pairs matter.** Where you have a choice, connect lower-wattage inverters to the Shellys that switch most frequently (e.g. the first plug in the best-fit order). The cycle counter and lifespan estimate help identify which plugs switch most.

> **Tip:** If relay longevity is a priority, prefer several lower-wattage inverters (e.g. 4 × 400 W) over fewer higher-wattage ones (e.g. 2 × 800 W). The total switching wear is distributed and the per-relay arcing energy is lower.

---

## 10  API Reference

All endpoints are HTTP/1.1 on port 80. No authentication required.

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/` | Serves `index.html` from LittleFS |
| GET | `/logo` | Returns the monkey JPEG logo |
| GET | `/api/status` | Full system status JSON |
| GET | `/api/config` | Config JSON — passwords not returned |
| POST | `/api/config` | Save new config, restart after 400 ms |
| GET | `/api/plug?idx=N&on=1` | Force plug N on (`on=1`) or off (`on=0`). Bypasses rate limiting. |
| GET | `/api/log` | Plain text log in `[Xs]` format |
| GET | `/api/heap` | Heap history: `{count, current, data:[…]}` (KB×10) |
| POST | `/api/reboot` | Reboots the device. Config is preserved. |
| POST | `/api/reset` | Factory reset: deletes `config.json`, reboots into AP mode. |

### 10.1  `/api/status` response fields

| Field | Description |
|-------|-------------|
| `gridPower` | Instantaneous grid power from last MQTT message (W) |
| `smoothPower` | Moving average over `avgWindowSec` (W) |
| `dynThresh` | Effective threshold this cycle (W) |
| `slotWh` | Accumulated export energy in current slot (Wh) |
| `expThresh` | Configured export threshold (W) |
| `mqttOk` | Boolean: MQTT broker connected |
| `mqttStale` | Boolean: no MQTT message for >60 s |
| `mqttAge` | Seconds since last MQTT message (−1 if never received) |
| `uptime` | Seconds since boot |
| `freeHeap` | Available heap in bytes |
| `version` | Firmware version string |
| `ip` / `rssi` / `ssid` | Network info |
| `plugs[N].cycles` | Relay switch count since boot |
| `plugs[N].lifeYrs` | Estimated remaining relay life in years (string, e.g. `"4.21"`) |
| `plugs[N].fault` | Boolean: plug has not reached commanded state |
| `plugs[N].nomW` | Configured nominal power (W) |
| `plugs[N].power` | Last measured power (W) |
| `plugs[N].on` | Boolean: last confirmed relay state |
| `plugs[N].online` | Boolean: polled successfully in last 30 s |
| `ctrlInterval` | Control interval in seconds (used by UI to match refresh rate) |
| `onThresh` | Configured ON threshold (W) |
| `avgWin` | Effective average window in seconds |

---

## 11  Troubleshooting

### 11.1  Plug won't switch

**Force ON / Force OFF button has no effect**

- *Plug offline:* Check the Online indicator. If offline, verify the Shelly is powered and on the same Wi-Fi.
- *Wrong IP:* Open `http://<shelly-ip>` directly in a browser to confirm the Shelly is reachable at that address.
- *Wrong generation:* Gen 1 uses `/relay/0`; Gen 2/3 uses `/rpc/Switch.Set`. The wrong setting causes HTTP 404 — check the Serial log.

**Control algorithm switches plug but it doesn't respond**

- *Rate-limited:* The last toggle may have been too recent. Check `minToggleSec` / `minToggleOffSec`. The log will say `rate limited`.
- *Fault flag set:* If the FAULT indicator is shown, use Force ON/OFF to reset.

### 11.2  Web UI unreachable

**Browser cannot connect to `http://<device-ip>`**

- *Device in AP mode:* If Wi-Fi credentials are wrong, the device falls back to AP mode (SSID: FartNSpark, IP: 192.168.4.1). The AP password is printed in the Serial boot log. **Disable mobile data on your phone before connecting** — iOS/Android will route traffic via LTE if the AP has no internet.
- *IP changed:* Check your router's DHCP table. Assign a static lease to the device's MAC address to avoid this.
- *Device crashed:* Check the Serial Monitor. If heap drops below 8 kB the device auto-reboots.

**Page loads but shows old content after firmware update**

- Press Ctrl+Shift+R (Cmd+Shift+R on Mac) to force a hard reload.
- If you built a custom firmware without incrementing `FW_VERSION`, the old HTML will be served from LittleFS cache.

### 11.3  MQTT not connecting

**MQTT badge shows DISCONNECTED**

- *Wrong host/port:* Verify `broker.hivemq.com:1883` is reachable, or switch to a local Mosquitto instance.
- *Broker requires auth:* Enter username and password in the MQTT section.
- *Port blocked:* Some networks block outbound port 1883. Try a local broker.

**MQTT connects but grid power never updates**

- *Wrong topic:* Use MQTT Explorer or `mosquitto_sub` to verify what your WattWächter publishes.
- *Wrong power key:* Check the Power Key field. The firmware tries common fallback keys before giving up — see [section 12.2](#122-power-key-resolution-order).
- *Payload format mismatch:* See [section 12](#12-mqtt--wattwächter-integration) for supported formats.

### 11.4  Power readings wrong or zero

**Plug reports 0 W but inverter is producing**

- *Poll frequency:* Each plug is polled once per second round-robin. With 8 plugs, each is polled every ~8 s — this is normal.
- *Gen mismatch:* Gen 2 uses flat JSON (`doc["apower"]`, `doc["output"]`). Gen 1 uses nested JSON (`doc["meters"][0]["power"]`). Check the generation setting.
- *Verify manually:* Open `http://<shelly-ip>/rpc/Switch.GetStatus?id=0` (Gen 2) or `http://<shelly-ip>/status` (Gen 1) in a browser and inspect the JSON.

### 11.5  System switches too frequently / oscillates

- *Average window too short:* Increase `avgWindowSec` (try 180–300 s). See section 2.2 for simulation data.
- *Post-OFF lockout too short:* Increase `offLockoutSec` to match or exceed the inverter ramp-up time.
- *nomW not configured:* Without `nomW`, the keep-strongest rule is disabled and best-fit selection falls back to fleet average — less accurate. Enter each inverter's rated power.

### 11.6  Heap keeps decreasing / device reboots

- The `logBuffer` String is capped at 1200 bytes — small fluctuations in the heap sparkline are normal.
- If free heap drops below 8 kB, the device auto-reboots as a safety measure. This is intentional and the system resumes normally after reboot.
- A sustained decline over days may indicate a fragmentation issue — contact the developer.

---

## 12  MQTT & WattWächter Integration

The firmware subscribes to a single MQTT topic and extracts the grid power value from the JSON payload.

### 12.1  Standard Tasmota payload

Default WattWächter payload format:

```json
{
  "Time": "2024-01-15T14:23:05",
  "SML": { "grid_power": 342.5 }
}
```

Config: Topic = `tele/WattWaechter/SENSOR`, Power Key = `grid_power`

### 12.2  Power key resolution order

The firmware tries to find the power value in this order:

1. `SML.<mqttPowerKey>` — e.g. `SML.grid_power`
2. `root.<mqttPowerKey>` — flat top-level key
3. Common fallback keys: `grid_power`, `Power`, `power`, `P_total`, `Wirkleistung`, `16.7.0`, `1.7.0`
4. `ENERGY.Power` — Tasmota energy module format
5. `root.<mqttPowerKey>` — second attempt at flat key

### 12.3  WattWächter simulator

The project includes a Python simulator for testing without real hardware.

```bash
pip install paho-mqtt
python wattwachter_sim.py
```

| Key | Action |
|-----|--------|
| `s` | Cycle through test power levels: −600, −300, 0, 300, 700, 900 W |
| `e` | Enter a custom power value manually |
| `x` | Exit |

Publishes to `tele/WattWaechter/SENSOR` on `broker.hivemq.com:1883`.

### 12.4  Using a local Mosquitto broker

For production use, a local MQTT broker is strongly recommended over the public `broker.hivemq.com`.

```bash
# Install on Raspberry Pi / Ubuntu
sudo apt install mosquitto mosquitto-clients
sudo systemctl enable mosquitto

# Verify
mosquitto_sub -h localhost -t tele/WattWaechter/SENSOR
```

Then set the MQTT Host in Configuration to the broker's LAN IP address (e.g. `192.168.1.10`).

---

## 13  Appendix A — Factory Reset

A factory reset clears `config.json` and `html.ver` from LittleFS and reboots into AP mode. All settings (Wi-Fi, MQTT, plug configuration, AP password) are erased.

**Method 1 — Via Web UI**

Go to the **Configuration** tab and click **⚠ Factory Reset**. Confirm the dialog. The device reboots into AP mode immediately.

**Method 2 — Via Serial**

Connect the Wemos via USB, open Serial Monitor at 115200 baud, and send:

```
RESET
```

The current AP password is also printed on every boot:
```
[AP] SSID: FartNSpark Password: <your-password>
[AP] Send 'RESET' via Serial to clear config if locked out.
```

**Method 3 — Re-flash**

Flash the firmware again. On first boot with no `config.json`, the device starts in AP mode automatically.

---

## 14  Appendix B — Firmware Version History

| Version | Key changes |
|---------|-------------|
| v2.001 | Baseline: boot fix, best-fit ON/OFF switching, LittleFS HTML caching |
| v4.000 | Moving average (180 s), 3-mode MQTT failsafe, `nominalW` per plug, never-off-on-import rule, Soll/Ist fault detection, energy budget |
| v4.001 | `plugEstW()` priority: measured first, `nomW` as fallback, fleet average last |
| v4.002 | Post-OFF lockout (`offLockoutSec`, default 90 s), keep-strongest-suitable-inverter rule in OFF branch |
| v4.003 | Relay cycle counter per plug (RAM-only, resets on reboot), lifespan estimate on plug cards, configurable `relayRatedCycles` (default 100,000) |
| v4.004 | Display of instantaneous vs averaged grid power in Grid Power card; `avgWin` in status JSON; `fabsf()` fix for Shelly apower sign |
| v4.005 | On/Off Cycles label (each op = 0.5 cycles); configurable ON threshold (`onThreshold`, default 0 W); AP mode stability (`apMode` flag, guards WiFi/MQTT/control/async state machines in `loop()`); factory reset via web UI (`/api/reset`) and Serial `RESET` command |
| v4.006 | Fix `avgGridPower()` ring buffer — was always reading slots 0–35 regardless of write pointer; plug card: Est. Lifetime label, muted cycle count color, 1.5px border badge; rename Reset → Discard Changes; Reboot button + `/api/reboot` endpoint; NTP guarded behind `!apMode` |
| v4.007 | Post-ON lockout (`onLockoutSec`, default 90 s) — prevents back-to-back inverter switching during ramp-up |
| v4.008 | UI refresh interval dynamically matches `ctrlIntervalSec` — no longer hardcoded at 10 s |
| v4.009 | ⚠ FAULT badge on plug cards; faulted plugs excluded from ON and OFF candidate selection; `faultTimeoutSec` default raised 30 → 60 s; `onLockoutSec` default lowered 120 → 90 s; AP password configurable, printed to Serial on boot, red warning in config UI while default `deye1234` is in use |
| v4.010 | Heap monitor stores rolling hourly minimum instead of spot reading (7-day low now meaningful); config UI label renames: Control Interval → Control & Screen Update Interval; Min Toggle Interval → Min time between ON commands (per plug); Min OFF Interval → Min time between OFF commands (per plug); Fault Timeout → Relay state error confirmed after; Post-ON/OFF Lockout labels clarified as across all plugs |
| v4.011 | Fix false faults on rate-limited commands: `desiredState` and fault timer now set in `asyncSwitchTick` SS_SENDING (when HTTP command actually fires) instead of `queueSwitch` (when merely queued). Eliminates spurious FAULT badges after Force OFF/ON while per-plug rate limit is still active. |

---

*Fart 'N Spark Solar Export Controller — Firmware v4.011 — Open-source firmware, no warranty. Install by qualified personnel only.*
