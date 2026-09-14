# RoboGuide — Upgrade Build Notes

Upgrade of the original IntelliRock prototype, built for WRO 2026 (Future
Innovators — "Robots Meet Culture", Area 1: Protecting, Preserving & Sharing
Cultural Heritage).

## What's new vs the IntelliRock prototype

| Feature | IntelliRock (before) | RoboGuide (after) |
|---|---|---|
| Proximity response | IR sensor → buzzer ON/OFF | Ultrasonic sensor → buzzer intensity scales continuously with distance |
| Language selection | Manual button (planned) | Gesture wave detected by IR sensor — no moving parts to wear out |
| Visitor data | None | Daily + lifetime visitor counter, debounced, stored locally |
| Learning platform | QR code (not yet built) | QR code links to a page hosted directly by the Pico W's own WiFi hotspot — works with **zero internet access** |
| AI chatbot | — | Works online when available; offline, full text content still works |

## Wiring assumptions

- OLED (SSD1306, I2C): SDA → GP0, SCL → GP1
- Ultrasonic (HC-SR04): TRIG → GP2, ECHO → GP3
- IR sensor: GP4
- Buzzer (PWM-capable): GP5

If your actual wiring is different, just change the pin numbers at the top of
`main.py` — nothing else in the code needs to change.

## Libraries you need on the Pico before running this

1. **`ssd1306.py`** — the standard MicroPython SSD1306 OLED driver.
   Widely available from the official micropython-lib / Adafruit repos —
   search "micropython ssd1306.py" and copy it onto the Pico alongside
   `main.py`.
2. **`uQR.py`** (optional, for the QR code on the OLED) — a small
   pure-Python QR code generator for MicroPython, e.g. the
   `micropython-uQR` project. If you don't install this, the code still
   runs fine — it just shows the WiFi name and URL as text instead of a
   QR graphic, so the unit never breaks in the field without it.

## How the offline learning platform works

1. The Pico W starts its own WiFi hotspot (default name `RoboGuide-Learn`,
   password `learnrockart` — change both in the CONFIG section).
2. After the slideshow finishes cycling through all languages, the OLED
   shows a QR code (or fallback text) pointing visitors to connect to that
   hotspot and open a page at the Pico's own IP address.
3. That page is served directly by the Pico itself — no internet needed.
4. If/when the site does have real internet access, you can extend this to
   detect that and redirect to the full online platform with the AI
   chatbot. Offline, the core educational content always still works.

## Things to tune for your actual site

- `DIST_WARN_START` / `DIST_DANGER` — measured in cm, adjust based on how
  far the real rock art wall is from the RoboGuide unit.
- `SLIDES` — add/edit the language text here; keep each line under ~16
  characters per wrapped line for the small OLED.
- `AP_PASSWORD` — must be 8+ characters for MicroPython's AP mode to accept it.

## Known limitations (worth mentioning to WRO judges honestly)

- The Pico W has no real-time clock battery by default, so "daily" counts
  are based on time since boot, not a real calendar date, unless you sync
  time via NTP whenever the unit does have internet access.
- The AI chatbot itself needs real compute and can't run on the Pico W —
  it's designed as an online-only enhancement layered on top of the
  always-available offline content.
