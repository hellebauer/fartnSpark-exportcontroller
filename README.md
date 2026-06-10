# Fart 'N Spark ⚡

**Solar export controller for ESP8266 (Wemos D1 Mini) + Shelly plugs**

Reads grid power from a WattWächter energy meter via MQTT and automatically switches PV microinverters on and off to keep grid export below a configurable threshold. Up to 8 Shelly plugs (Gen 1/2/3) supported.

---

## What it does

- Subscribes to an MQTT topic for live grid power (W)
- Maintains a moving average to filter out appliance and cloud shadow noise
- Switches inverters ON when importing, OFF when exporting above threshold
- Best-fit plug selection — one plug per cycle, closest match to available headroom
- Fault detection — flags plugs that don't respond to commands
- AP fallback — if Wi-Fi fails, starts a config access point at `192.168.4.1`
- Web UI with status, log, heap monitor, and full configuration

## Hardware

- Wemos D1 Mini (ESP8266, 4 MB flash)
- Shelly Plug S / Plus Plug S (Gen 1, 2, or 3)
- WattWächter (Tasmota energy meter publishing MQTT)

## Quick start

1. Flash `deye_shelly_8266V1_v4010.ino` via Arduino IDE
2. Connect to Wi-Fi AP **FartNSpark** — check Serial Monitor for password
3. Browse to `http://192.168.4.1` — enter Wi-Fi and MQTT settings
4. Add your Shelly plug IPs and inverter nominal power
5. Done — the controller starts automatically on first MQTT message

> ⚠️ Mains voltage is involved. Installation by a qualified electrician only. See the full manual for safety and legal notices.

## Docs

See [FartNSpark_Manual_v4010.md](FartNSpark_Manual_v4010.md) for full documentation including configuration reference, algorithm description, API, and troubleshooting.

---

*Open-source, no warranty. Use at your own risk.*

---

## Disclaimer

This firmware is provided for educational and experimental purposes only. It involves control of mains-voltage equipment (230 V AC). The author accepts no liability for damage to property, injury, or death resulting from its use. Installation must be carried out by a qualified electrician in accordance with all applicable local regulations. Use entirely at your own risk.

See [FartNSpark_Manual_v4010.md](FartNSpark_Manual_v4010.md) for the full safety and legal notice before installing.

## License

MIT © hellebauer
