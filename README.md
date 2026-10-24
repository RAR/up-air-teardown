# UniFi UP-Air Quality — Teardown & Notes

Reverse-engineering notes for the **Ubiquiti UniFi Protect Air Quality** sensor
(model `UP-AirQuality`), an ESP32-based wired-Ethernet air sensor.

> Everything here was gathered from **my own device on my own LAN**, read-only,
> over the serial service port. Nothing was reflashed or modified. Unit-specific
> identifiers (MAC host bytes, LAN IP, serial) have been scrubbed.

## TL;DR

- **MCU:** Espressif ESP32-D0WD-V3 (rev v3.1), 40 MHz crystal
- **Flash:** GigaDevice GD25Q128E — 16 MB SPI NOR
- **PSRAM:** ESP-PSRAM64H — 8 MB
- **Ethernet PHY:** Realtek RTL8201F 10/100 (wired-only; no Wi-Fi)
- **Air sensor:** Sensirion **SEN66** over I²C (PM1/2.5/4/10, VOC, NOx, CO₂, RH, temp) + a vape-detection threshold
- **Firmware:** `UPAQ.esp32.v1.0.16.0` on ESP-IDF v5.5.4
- **Network:** only **TCP 8080** open — HTTPS/TLS1.3 UniFi inform channel. No open REST for readings.
- **Service port:** 6-pin 0.5 mm FFC (`J6`) → full ESP-IDF serial console (`esp>`)
- **Security:** Secure Boot V1 **enabled**, Flash Encryption **enabled** (dev mode). Don't reflash.

## What's in here

| Doc | Contents |
|-----|----------|
| [HARDWARE.md](HARDWARE.md) | Chip IDs, **board photos**, the FFC service port, UART pins |
| [SERIAL-ACCESS.md](SERIAL-ACCESS.md) | How to reach the console (Linux gotchas, baud, ModemManager) |
| [CONSOLE.md](CONSOLE.md) | Full console command reference |
| [SECURITY.md](SECURITY.md) | ESP32 secure-boot / flash-encryption posture; why not to reflash |

## Getting air-quality data out of it

Three routes, easiest first:

1. **UniFi Protect controller** — the device reports to your console; read it back with
   the [`uiprotect`](https://github.com/uilibs/uiprotect) Python library or the
   Home Assistant UniFi Protect integration. This is the intended path and needs no hardware.
2. **Local serial console** — tap `J6`, connect the SEN66 head, run `i2c_scan` (expect
   `0x6B`) then `sen66_fixed` for live readings. See [SERIAL-ACCESS.md](SERIAL-ACCESS.md).
3. **HTTPS 8080** — the device's own management channel; controller-authenticated, not a
   convenient data source.

## Status

Hardware path characterized and shelved; primary data path is the local Protect controller.
Contributions / corrections welcome — this is one unit, one firmware rev.
