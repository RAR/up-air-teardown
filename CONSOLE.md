# Console Command Reference

Full ESP-IDF console command set, from `help` and `help -v 1` (identical — nothing
hidden). There is **no** `ssh` / `nvs` / `telnet` / `debug` command; the local UART
console is the only shell.

## Sensors

| Command | What it does |
|---------|--------------|
| `sen66_fixed` | **Live telemetry** — PM/VOC/NOx/CO₂/RH/temp (sensor head must be connected) |
| `sen66 -spafdc` | SEN66 raw/flagged read |
| `sen66_temp` | SEN66 temperature |
| `i2c_scan` | Scan the I²C bus (expect SEN66 at `0x6B`) |
| `i2c_get` / `i2c_set` | Read / write an I²C register |
| `adc_get` | Read an ADC channel |

## GPIO / peripherals

`gpio`, `pwm_set_duty`

## Network

| Command | What it does |
|---------|--------------|
| `ethernet info` / `stop` / `start` / `dhcp` / `static` | Ethernet control |
| `ping` | ICMP ping |
| `iperf` | Throughput test |

## Filesystem

| Command | What it does |
|---------|--------------|
| `ls` / `cat` / `rm` | Browse / read / **delete** files on `/storage` |
| `partitions` | Flash partition table |

> ⚠️ `rm` deletes live files. The httpd TLS private key/cert live on this FS — don't
> dump secrets into a shared log.

## Info / diagnostics

`info`, `platform_info`, `uniqueid`, `free`, `uptime`, `ps`, `tasks`

## Lights & display

`ringled`, `ringled_brightness`, `ringled_metric`, `led_state`, `led_status`, `night_mode`

## Config & system

| Command | What it does |
|---------|--------------|
| `console on` / `off` / `status` | Toggle the console |
| `log_level` | Set log verbosity |
| `timezone` | Set timezone |
| `vape_sensitivity_thresh` | Vape-detection threshold |
| `system_save_cfg` / `system_load_cfg` | Persist / load config (`/storage/www/system.cfg`) |
| `time_sync` | Sync clock |
| `reboot` | **Soft restart** |
| `reset` | ⚠️ **FACTORY reset** — un-adopts the device. Use `reboot` for a normal restart. |

## HTTPS API surface (TLS on 8080)

Routes seen in firmware: `/api/cmd`, `/api/ota`, `/api/reboot`, `/api/reset`, `/api/*`.
UCPV4 operations: `reboot`, `factoryReset`, `updateFirmware`, `locate`, `getInfo`,
`getSupportInfo`, `networkStatus`, `setInfo`, `setIndicatorLed`, `liveLogging`,
`sshService`, `changeUserPassword`, `changeLightSettings`, `setConfig`,
`setRingLedColor`. MCAD UDP discovery on port `10001`.

`sshService` is a **controller-side** UCP operation, not a local command — you can't
turn on SSH from the device (and there's no sshd to turn on). Boot also probes two NVS
flags that are unset by default: `console_enable` and `reset_enable`.
