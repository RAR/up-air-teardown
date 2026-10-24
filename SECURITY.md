# Security Posture

Read from the boot log and the ESP32's fuse-enforced features. **Don't reflash this
device** — the combination below makes a bad flash a likely brick, with no upside.

All of this is enforced by internal **eFuses (OTP)** — one-time-programmable bits inside
the ESP32, *not* board straps or resistors. Permanent, irreversible, and the keys are
read-protected (unreadable).

## Secure Boot V1 — ENABLED

Every boot the ROM/bootloader verifies an **ECDSA signature** over the app image.
Unsigned or modified firmware is **rejected**. You cannot run custom firmware without
Ubiquiti's private signing key.

## Flash Encryption — ENABLED (Development mode)

- The boot log reports flash encryption in **DEVELOPMENT** mode:
  `W flash_encrypt: Flash encryption mode is DEVELOPMENT (not secure)`.
- "Development" means the `FLASH_CRYPT_CNT` counter still has **plaintext flashes left**
  (this unit: 3). Release mode maxes that counter and burns
  `DISABLE_DL_DECRYPT` / `DISABLE_DL_ENCRYPT` / `DISABLE_DL_CACHE`.
- Because `DISABLE_DL_DECRYPT` is **not** burned, plaintext firmware *is* extractable via
  the UART download-decrypt path. A plain `read_flash`, by contrast, returns **ciphertext**.
- At runtime the app sees decrypted data, so `cat`-ing files over the console works fine.

Likely left in dev mode for manufacturing / RMA flexibility or early firmware.

## Why not to reflash

1. Secure Boot will reject anything you sign yourself.
2. Each plaintext write burns one of a small, finite counter — exhaust it and the device
   can refuse to boot.
3. There's no functional gain: you already have a full console and can read the
   filesystem live.

## Inspecting fuses (read-only)

If you want to confirm the posture yourself, **read only** — never `burn` anything:

```bash
espefuse.py --port /dev/ttyACM0 summary
```

Look at `FLASH_CRYPT_CNT`, `ABS_DONE_0` (secure boot), the `DISABLE_DL_*` bits, and
`JTAG_DISABLE`.
