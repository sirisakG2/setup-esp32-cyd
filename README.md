# Setup: ESP32-2432S028 ("Cheap Yellow Display" / CYD)

A log of the problems hit (and fixes found) while bringing up an ESP32-2432S028R
"Cheap Yellow Display" board on macOS with PlatformIO + LovyanGFX — from USB
detection through display bring-up to touch bring-up and calibration.

Board: ESP32-D0WD-V3, 240x320 ILI9341 SPI display, resistive XPT2046 touch,
CH340C USB-UART bridge.

See the [Issues](../../issues) tab for each individual problem, its root cause,
and the fix.

The working firmware this troubleshooting led to lives at
[sirisakG2/esp32-cyd-macropad](https://github.com/sirisakG2/esp32-cyd-macropad).
