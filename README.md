# Fart 'N Spark ⚡

**Solar export controller for ESP8266 (Wemos D1 Mini) + Shelly plugs**

Reads live grid power from any MQTT-enabled smart energy meter (WattWächter, Tasmota, Shelly 3EM, Deye, Home Assistant bridge, and others) and automatically switches PV microinverters on and off to keep grid export below a configurable threshold. Up to 8 Shelly plugs (Gen 1/2/3) supported.

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
- Any smart energy meter that publishes grid power via MQTT — WattWächter, Tasmota-based meters, Shelly 3EM, Deye inverters, Home Assistant MQTT bridge, and others. Configure the MQTT topic and JSON power key in the web UI.

## Quick start

1. Flash `deye_shelly_8266V1_v4011.ino` via Arduino IDE
2. Connect to Wi-Fi AP **FartNSpark** — check Serial Monitor for password
3. Browse to `http://192.168.4.1` — enter Wi-Fi and MQTT settings
4. Add your Shelly plug IPs and inverter nominal power
5. Done — the controller starts automatically on first MQTT message

> ⚠️ Mains voltage is involved. Installation by a qualified electrician only. See the full manual for safety and legal notices.

## Docs

See [FartNSpark_Manual_v4011.md](FartNSpark_Manual_v4011.md) for full documentation including configuration reference, algorithm description, API, and troubleshooting.

---

*Open-source, no warranty. Use at your own risk.*

---

## Disclaimer

This firmware is provided for educational and experimental purposes only. It involves control of mains-voltage equipment (230 V AC). The author accepts no liability for damage to property, injury, or death resulting from its use. Installation must be carried out by a qualified electrician in accordance with all applicable local regulations. Use entirely at your own risk.

See [FartNSpark_Manual_v4011.md](FartNSpark_Manual_v4011.md) for the full safety and legal notice before installing.

## License

MIT © hellebauer

## Third-party acknowledgements

This project uses the following open-source libraries:

| Library | Author | License |
|---------|--------|---------|
| [ArduinoJson](https://arduinojson.org) | Benoit Blanchon | MIT |
| [PubSubClient](https://github.com/knolleary/pubsubclient) | Nick O'Leary | MIT |
| [ESPAsyncWebServer](https://github.com/me-no-dev/ESPAsyncWebServer) | me-no-dev | LGPL 3.0 |
| [ESPAsyncTCP](https://github.com/me-no-dev/ESPAsyncTCP) | me-no-dev | LGPL 3.0 |
| [ESP8266 Arduino Core](https://github.com/esp8266/Arduino) | ESP8266 Community | LGPL |
| [LittleFS](https://github.com/littlefs-project/littlefs) | ARM Limited | BSD 3-clause |

LGPL libraries are used unmodified. Source code is available in this repository, satisfying LGPL requirements.

