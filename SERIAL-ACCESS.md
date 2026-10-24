# Serial Access

The `J6` FFC port exposes the ESP32's UART console. Tap TX/RX/GND with a 3.3 V
USB-to-TTL adapter (see [HARDWARE.md](HARDWARE.md) for pins). **RX ↔ TX crossed.**

## Linux gotchas (these cost real time)

1. **The adapter may enumerate as `/dev/ttyACM0`, not `/dev/ttyUSB0`.**
   Some USB-TTL bridges present as CDC-ACM. Check `dmesg` right after plugging in.

2. **Stop ModemManager** — on a CDC-ACM device it grabs the port and injects AT probes:
   ```bash
   sudo systemctl stop ModemManager
   # if it keeps coming back:
   sudo systemctl mask ModemManager
   ```

3. **Garbage characters = wrong baud, right port.** If the RX LED blinks on boot but you
   see `���…`, you're connected — just sweep the baud rate.

## Baud rates

- **ROM download/boot banner:** appears at **74880** baud (ESP32 quirk).
- **Main application console:** the rate that renders clean text (sweep 115200 / 74880 / others).
- `esptool` auto-syncs regardless of terminal baud.

## Viewing the console

```bash
# screen
screen /dev/ttyACM0 115200

# pyserial miniterm
python3 -m serial.tools.miniterm /dev/ttyACM0 115200

# raw hexdump (useful while hunting for the right baud)
stty -F /dev/ttyACM0 115200 raw -echo
cat /dev/ttyACM0 | xxd
```

Success looks like an interactive `esp>` prompt with TAB-completion and history.
See [CONSOLE.md](CONSOLE.md) for commands.

## Optional: bridge the console over the LAN

There's no SSH/telnet on the device (it's an RTOS, not Linux). To get a remote shell,
bridge the UART from an attached SBC:

```bash
socat TCP-LISTEN:2323,reuseaddr,fork /dev/ttyACM0,b115200,raw,echo=0
# then from another host:
telnet <sbc-ip> 2323
```
