# VoltGuard

An authentication-gated portable power bank. An ESP32 running MicroPython verifies a PIN entered on a capacitive touch keypad and controls a USB power switch, so the output stays disabled until authorization succeeds.

> **Status:** Work in progress. Currently at the breadboard prototype stage (Stage 1).

## Concept

Most power banks deliver power to anything that is plugged in. VoltGuard places an authentication gate between the battery and the USB output:

```
Battery -> charge/protection board -> 5V boost -> [ USB power switch ] -> USB-A output
                                                          ^
                                                        ESP32
                                                    ^           ^
                                          Touch keypad       OLED display
```

The switch is off by default. The ESP32 enables it only after a correct PIN, and disables it again after a timeout or when the device is unplugged.

## Features

| Feature | Status |
|---|---|
| OLED status display | In progress |
| Capacitive touch keypad (MPR121) | In progress |
| PIN entry with masked digits | Planned |
| Salted SHA-256 PIN hash stored in flash | Planned |
| Lockout with growing delays, counter persisted across power cycles | Planned |
| Gated USB-A output with auto-lock timeout | Planned |
| Deep sleep while locked, wake on button or touch | Planned |
| Battery percentage readout (MAX17048) | Planned |
| Optional fingerprint unlock | Considered |

## Hardware (breadboard stage)

- ESP32 DevKit (classic, WROOM-32)
- MPR121 capacitive touch breakout
- 0.96" SSD1306 I2C OLED display
- Two tactile buttons (Enter, Wake)
- USB power switch (TPS2051-type breakout, or P-channel MOSFET with NPN driver)
- LED with 330 ohm resistor for testing the lock logic
- Breadboard, jumper wires, 10 kOhm resistors

The prototype runs on USB power. The battery, charge board and boost converter are added in a later stage.

## Wiring (Stage 1)

| Device | Pin | ESP32 |
|---|---|---|
| OLED | VCC / GND | 3V3 / GND |
| OLED | SDA / SCL | GPIO21 / GPIO22 |
| MPR121 | VIN / GND | 3V3 / GND |
| MPR121 | SDA / SCL | GPIO21 / GPIO22 |

Both devices share the I2C bus. Use 3.3V only.

## Software setup

1. Install [Thonny](https://thonny.org).
2. Flash MicroPython onto the ESP32 (Thonny: Tools, Options, Interpreter, "Install or update MicroPython").
3. Install the OLED driver: Tools, Manage packages, `micropython-ssd1306`.
4. Save `main.py` to the device.

On startup the shell prints the detected I2C devices. Expect `0x3c` (OLED) and `0x5a` (MPR121).

## Repository structure

```
VoltGuard/
  main.py      Stage 1: OLED text and keypad touch reading
  README.md
```

## Roadmap

1. OLED text and keypad reading
2. PIN entry (masked digits, Enter, clear)
3. Salted-hash PIN verification
4. Gated output with auto-lock timeout
5. Lockout with persistent failed-attempt counter
6. Deep sleep and wake handling
7. Battery, charge board and boost converter integration
8. Enclosure and security testing

## Security notes

- The output switch is designed to fail closed: a pull-down keeps it off if the microcontroller crashes or loses power.
- The PIN is never stored in plain text.
- The failed-attempt counter must be stored in flash, not RAM, otherwise cutting power resets the lockout.
- A short PIN has a small search space. If an attacker can read the flash, a hash alone will not protect it. ESP32 flash encryption and secure boot are planned for hardening.
- Do not commit real PINs, hashes or salts to this repository.

## License

To be decided.
