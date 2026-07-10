# Embedded Systems Learning & Project Portfolio

A structured, hands-on path through embedded systems — six focused projects that build on each other, capped by an integrated capstone. Each project lives in its own folder and produces working, demonstrable artifacts (not just notes).

> 📋 The detailed phase-by-phase roadmap with milestones, deliverables, and resources is in **[PLAN.md](PLAN.md)**.

## The Projects

| # | Folder | Topic | Core Question It Answers |
|---|--------|-------|--------------------------|
| 1 | [embedded-fundamentals](https://github.com/BlindinShadow/embedded-fundamentals) | Embedded Systems Foundations | How do computers-inside-things differ from PCs, and how do I talk to hardware from C? |
| 2 | [firmware-baremetal](https://github.com/BlindinShadow/firmware-baremetal) | Firmware Development (Bare-Metal) | What actually happens from power-on to `main()`, and how do I control hardware with zero libraries? |
| 3 | [arm-cortex](https://github.com/BlindinShadow/arm-cortex) | ARM Cortex-M Architecture | How does the Cortex-M core itself work — interrupts, exceptions, memory, low power, debug? |
| 4 | [embedded-software-eng](https://github.com/BlindinShadow/embedded-software-eng) | Embedded Software Engineering | How do professionals structure, test, and ship embedded code (drivers, HALs, state machines, CI)? |
| 5 | [embedded-rtos](https://github.com/BlindinShadow/embedded-rtos) | Real-Time Systems & RTOS | How do I run many time-critical things "at once" correctly, with FreeRTOS? |
| 6 | [embedded-iot](https://github.com/BlindinShadow/embedded-iot) | IoT & Connectivity | How do I get a device securely onto a network and into the cloud (MQTT, TLS, OTA)? |
| 7 | [sentinel](https://github.com/BlindinShadow/sentinel) | **Integrated Capstone** | Can I combine all of the above into one real product-grade device? |

**Recommended order:** 1 → 2 → 3 → 4 → 5 → 6 → 7. Projects 2–3 are deliberately sequenced before 4: you learn the raw metal first, *then* how to engineer on top of it. See PLAN.md for what "done" means for each.

## The Capstone (Project 7)

**"Sentinel" — a smart environmental monitoring node.** A battery-conscious device that samples sensors (temperature/humidity/pressure/air quality), runs a FreeRTOS task architecture, displays locally, logs to flash, and publishes securely over MQTT to a live dashboard with OTA firmware updates. Every earlier project contributes a subsystem — details in [PLAN.md](PLAN.md#project-7--integrated-capstone-sentinel).

## Hardware

You can start **today with zero hardware** (simulators cover Projects 1–5), but real boards are strongly recommended from Project 2 onward.

| Item | Used In | Approx. Cost (India) | Notes |
|------|---------|----------------------|-------|
| **STM32 Nucleo-F446RE** | P2–P5, P7 | ₹2,000–2,800 | Primary board. Cortex-M4F, onboard ST-LINK debugger — no extra programmer needed. (Nucleo-F411RE is a fine cheaper substitute.) |
| **ESP32 DevKitC / DevKit v1** | P6, P7 | ₹400–700 | Wi-Fi + BT, the standard IoT learning platform. |
| **BME280 sensor module** | P4–P7 | ₹300–500 | Temp/humidity/pressure over I2C — the workhorse peripheral for driver-writing. |
| Breadboard, jumpers, LEDs, buttons, resistors | All | ₹300–500 | Basic kit. |
| SSD1306 0.96" OLED (I2C) | P4+, P7 | ₹200–350 | Local display. |
| *Optional:* logic analyzer (8ch, Saleae-clone) | P4+ | ₹700–1,200 | Seeing your I2C/SPI/UART on a timeline is a superpower. |

**No-hardware / pre-hardware options:** [Wokwi](https://wokwi.com) (simulates ESP32, RP2040, STM32 — excellent), QEMU (`qemu-system-arm`, Cortex-M3 targets), and [Renode](https://renode.io) (whole-board simulation, great for RTOS work).

## Toolchain (Linux / WSL2)

```bash
# Cross-compiler, debugger, flasher
sudo apt install gcc-arm-none-eabi gdb-multiarch openocd stlink-tools

# Build tools
sudo apt install make cmake ninja-build

# Serial terminal
sudo apt install picocom  # or minicom

# ESP32 (Project 6) — installed later via ESP-IDF's own installer
# Unit testing (Project 4) — Ceedling: gem install ceedling
```

> **WSL2 note:** USB devices (ST-LINK, serial ports) need `usbipd-win` to pass through from Windows: `winget install usbipd` on the Windows side, then `usbipd bind`/`usbipd attach --wsl`. This is a one-time setup covered in Project 2.

## Repository Layout

Each project folder is its **own GitHub repository** (linked in the table above). This parent repo tracks only the roadmap docs — the project folders are gitignored here and managed independently.

## Conventions

- Each project folder gets its own `README.md` (goals, wiring, build/run instructions) and is built incrementally in stages.
- Everything builds from the command line with `make` or CMake — IDEs optional, understanding mandatory.
- Code in C (C11) primarily; the capstone may add C++ or MicroPython tooling on the host side.
- Each project ends with a short write-up of what was learned — these become your portfolio.

## Status

| Project | Status |
|---------|--------|
| embedded-fundamentals | 🔲 Not started |
| firmware-baremetal | 🔲 Not started |
| arm-cortex | 🔲 Not started |
| embedded-software-eng | 🔲 Not started |
| embedded-rtos | 🔲 Not started |
| embedded-iot | 🔲 Not started |
| sentinel (capstone) | 🔲 Not started |
