# Changelog

All notable changes to Fart 'N Spark are documented here.

---

## v4.011
- **Fix: false faults on rate-limited commands** — `desiredState` and fault timer are now set in `asyncSwitchTick` at the moment the HTTP command actually fires (SS_SENDING), not in `queueSwitch` when the command is merely queued. Eliminates spurious ⚠ FAULT badges after Force OFF/ON while the per-plug rate limit is still active.

## v4.010
- **Heap monitor: rolling hourly minimum** — each hourly sample now stores the worst-case (minimum) heap observed during that hour rather than a spot reading. 7-day low chart is now meaningful and comparable to the live current reading.
- **Config UI label renames** for clarity:
  - `Control Interval` → `Control & Screen Update Interval`
  - `Min Toggle Interval` → `Min time between ON commands (per plug, in seconds)`
  - `Min OFF Interval` → `Min time between OFF commands (per plug, in seconds)`
  - `Fault Timeout` → `Relay state error confirmed after (seconds)`
  - `Post-ON Lockout` → clarified as across all plugs
  - `Post-OFF Lockout` → clarified as across all plugs

## v4.009
- **⚠ FAULT badge** on plug cards — red badge shown inline when a plug has not reached its commanded state within `faultTimeoutSec` seconds
- **Faulted plugs excluded from control** — ON and OFF candidate selection now skips faulted plugs in all three loops (keep-strongest, best-fit OFF, best-fit ON)
- **`faultTimeoutSec` default raised 30 → 60 s** — reduces false fault rate from slow-responding Shellys
- **`onLockoutSec` default lowered 120 → 90 s** — consistent with `offLockoutSec`
- **AP password configurable** — new `apPassword` field in config; printed to Serial on every boot alongside a `RESET` command reminder; red warning shown in config UI while default `deye1234` is still in use

## v4.008
- **UI refresh interval matches control interval** — `setInterval` is now dynamic, reads `ctrlInterval` from `/api/status` on first load and adjusts automatically. No longer hardcoded at 10 s.

## v4.007
- **Post-ON lockout** (`onLockoutSec`, default 90 s) — after any ON command, the ON branch is blocked globally for this duration. Prevents back-to-back inverter switching before the first inverter has ramped up.

## v4.006
- **Fix: `avgGridPower()` ring buffer** — was always reading slots 0–35 regardless of write pointer position. Now walks backwards from `gridPowerBufIdx` using `(gridPowerBufIdx - 1 - i + 60) % 60`. Affected both display and control decisions.
- **Plug card badge restyling** — 1.5px bordered box around cycle/lifetime counter; lifetime value same font size as cycle count
- **`Est. Lifetime` label** added below lifetime value on plug cards
- **Rename Reset → Discard Changes** in config tab
- **Reboot button** added to config tab; new `/api/reboot` endpoint reboots without clearing config
- **NTP guarded behind `!apMode`** — no NTP requests sent when running as access point

## v4.005
- **On/Off Cycles** — each relay operation (ON or OFF) counts as 0.5 cycles; label updated from `cycles` to `On/Off Cycles`
- **Configurable ON threshold** (`onThreshold`, default 0 W) — replaces hardcoded two-branch ON logic; switch ON only when `smoothPower < onThreshold`
- **AP mode stability** — `apMode` flag gates WiFi reconnect, MQTT, `runControl()`, `asyncSwitchTick()`, `asyncPollerTick()` in `loop()`; prevents TCP timeouts and radio kicks in AP mode
- **Factory reset** — web UI button with confirmation dialog; `/api/reset` endpoint; Serial `RESET` command

## v4.004
- **Grid Power card** shows instantaneous (`now`) and averaged (`Xs avg`) grid power
- **`avgWin`** added to `/api/status` JSON
- **Fix: `fabsf()`** for Shelly `apower` sign (was using `abs()` which truncates floats)

## v4.003
- **Relay cycle counter** per plug (RAM-only, resets on reboot)
- **Lifespan estimate** on plug cards based on cycle rate and `relayRatedCycles`
- **`relayRatedCycles`** configurable (default 100,000)

## v4.002
- Post-OFF lockout (`offLockoutSec`, default 90 s)
- Keep-strongest rule — protects highest-power inverter from being shed
- Per-plug fault detection (`faultTimeoutSec`)
- Slot budget limiter (`slotBudgetWh`, `slotMinutes`)

## v4.001
- Moving average grid power (`avgWindowSec`, default 180 s)
- 3-mode MQTT failsafe (`mqttFailsafeMode`: all ON / S1 only / all OFF)
- `nominalW` per plug for best-fit estimation when measured power unavailable
- Dynamic threshold (`dynThresh`) based on slot budget

## v2.001
- Initial release
- Best-fit plug selection
- LittleFS HTML caching (cache keyed to firmware version)
- Gen 1 / Gen 2 / Gen 3 Shelly support
- Async HTTP state machines for switch and poll
- 7-day heap monitor
- Web UI: Status, Log, Configuration tabs
