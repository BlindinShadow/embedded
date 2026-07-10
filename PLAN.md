# Detailed Roadmap

Seven projects, roughly 26 weeks at a steady ~8–10 hrs/week pace (compress or stretch freely — milestones matter, dates don't). Each project lists **stages**, a **definition of done**, **deliverables**, and **key resources**.

**Guiding principle:** every stage produces something that *runs*. No stage is "read about X" — it's always "build X, and read what you need to build it."

---

## Project 1 — Embedded Fundamentals (`embedded-fundamentals/`)

**Theme:** What makes embedded different. C as it's actually used near hardware. No board required — host PC + Wokwi simulator.

**Duration:** ~2 weeks

### Stages

1. **Embedded-flavored C bootcamp** (host-based, with unit tests)
   - Bit manipulation library: set/clear/toggle/read bit fields, masks, shifts — the vocabulary of register access.
   - Fixed-width types (`uint8_t`…), `volatile`, `const` placement, storage classes, alignment, struct packing/padding, endianness detection.
   - Pointer drills: pointer-to-register simulation (`*(volatile uint32_t*)0x40020014`), function pointers (→ interrupt vector tables later), pointer arithmetic over buffers.
2. **Memory model deep-dive**
   - Write programs that deliberately place data in `.text` / `.data` / `.bss` / stack / heap; inspect with `objdump`, `nm`, `size`, `readelf`.
   - Why embedded avoids `malloc`; implement a simple static memory pool allocator.
3. **Core embedded data structures** (all statically allocated)
   - Ring buffer (the #1 embedded data structure — used for UART in every later project), simple scheduler table, debounce state machine.
4. **First "device" — simulated**
   - On Wokwi (Arduino Uno or Pi Pico): blink → button-controlled LED → traffic light state machine → 7-segment counter. Focus: reading a datasheet pinout, translating logic into GPIO operations.

### Definition of Done
- Bit-manipulation + ring-buffer libraries pass a unit-test suite on host.
- Can explain (in the project write-up) where every byte of a small C program lives in memory and prove it with binutils output.
- Traffic-light state machine runs on Wokwi.

### Deliverables
`bitops.c/h`, `ringbuf.c/h`, `pool_alloc.c/h` + tests · memory-layout lab notes · Wokwi project links · write-up.

### Key Resources
- *The C Programming Language* (K&R) as reference; **"Modern Embedded Systems Programming" YouTube course (Quantum Leaps / Miro Samek)** — lessons 1–15 pair perfectly with this project.
- `man` pages for `objdump`, `readelf`, `nm`.

---

## Project 2 — Bare-Metal Firmware (`firmware-baremetal/`)

**Theme:** Power-on to `main()` with your own hands. Zero vendor libraries — you write the startup code, linker script, and register definitions. **Board:** STM32 Nucleo-F446RE (QEMU/Renode as fallback).

**Duration:** ~4 weeks

### Stages

1. **Toolchain & flash pipeline**
   - Cross-compile a minimal binary; understand ELF vs `.bin` vs `.hex`; flash via OpenOCD/`st-flash`; set up WSL2 USB passthrough; first GDB session over ST-LINK (breakpoints, register inspection, memory watch).
2. **The boot sequence, hand-rolled**
   - Write your own **linker script** (FLASH/RAM regions, `.isr_vector`, `.text`, `.data` load-vs-run addresses, `.bss`).
   - Write your own **startup code**: vector table as an array of function pointers, `Reset_Handler` that copies `.data`, zeroes `.bss`, calls `main`.
   - Blink an LED using *only* the reference manual: enable GPIO clock via RCC, configure MODER, toggle ODR — every register address typed from the datasheet.
3. **Register-level peripheral drivers**
   - Define peripheral register structs (`typedef struct { volatile uint32_t MODER; … } GPIO_TypeDef;`) — build your own mini device header.
   - **SysTick** delay (first taste of the Cortex-M core peripherals) → **UART** transmit, then receive (polled) → `printf` retargeting over UART.
   - Clock tree: configure PLL to run at 180 MHz from the 16 MHz HSI; measure the difference.
4. **Interrupts, bare-metal**
   - EXTI button interrupt, UART RX interrupt feeding your Project-1 ring buffer, SysTick-based software timers. Understand NVIC enable/priority at the register level.
5. **Mini-bootloader (stretch goal)**
   - A tiny UART bootloader in the first flash sector that can receive and jump to an application image — demystifies how OTA will work in Project 6.

### Definition of Done
- A repo where `make flash` builds and flashes a multi-file firmware **containing zero vendor/HAL code** — your linker script, your startup, your register definitions.
- UART command console (e.g., `led on`, `stat`) driven by RX interrupts + ring buffer.
- Can walk through the boot sequence from vector-table fetch to `main()` on a whiteboard.

### Deliverables
Custom `linker.ld` + `startup.c` · GPIO/UART/SysTick register-level drivers · interrupt-driven UART console · Makefile + OpenOCD/GDB configs · write-up ("power-on to main(), explained").

### Key Resources
- **STM32F446 Reference Manual (RM0390)** + Nucleo-F446RE user manual — the primary texts.
- "Bare metal programming guide" by cpq (GitHub, STM32-based) — closest existing walkthrough to this project.
- *Beningo — "A step-by-step guide to writing a linker script"* style articles.

---

## Project 3 — ARM Cortex-M Deep Dive (`arm-cortex/`)

**Theme:** The core itself, not the peripherals around it. **Board:** same Nucleo. Uses CMSIS headers from here on (you've earned them by writing your own in P2).

**Duration:** ~3 weeks

### Stages

1. **Architecture on paper → in GDB**
   - Registers (R0–R15, xPSR, MSP/PSP, CONTROL), Thumb-2, the Cortex-M memory map, bit-banding. Verify each claim live in GDB.
   - Write and call small assembly routines from C (AAPCS calling convention); read the disassembly of your C code until it stops being scary.
2. **Exceptions & interrupts, properly**
   - Exception entry/exit (hardware stacking, `EXC_RETURN`), NVIC priority/grouping, preemption vs sub-priority — build a demo proving nested preemption.
   - **Fault handling:** deliberately cause HardFault/UsageFault/BusFault; write a fault handler that dumps the stacked frame and decodes the fault registers (CFSR). This becomes your debugging tool for every later project.
3. **Core peripherals & modes**
   - SysTick (revisit), PendSV (→ this is how RTOS context switching works — foreshadowing P5), SVC calls, privileged vs unprivileged, MSP vs PSP switching — write a toy "two-task switcher" using PendSV. **This is the single most valuable exercise in the whole plan.**
4. **Performance & power**
   - DMA: memory-to-memory then UART-TX-via-DMA; measure CPU relief.
   - Low-power modes (sleep/stop/standby), WFI, wake-on-interrupt; measure current if you have a meter.
   - FPU usage, cycle counting with DWT->CYCCNT; benchmark fixed-point vs float.

### Definition of Done
- Fault-handler library that prints a decoded crash report over UART (reused in P4–P7).
- Working two-context switcher built on PendSV + PSP (≤ ~150 lines) — you've essentially written an RTOS kernel's heart.
- Demo suite: nested interrupts, DMA UART, sleep-mode blinky.

### Deliverables
`fault_handler.c/h` · `toy_switcher/` · DMA + low-power demos · annotated notes on exception entry/exit · write-up.

### Key Resources
- **Joseph Yiu, *The Definitive Guide to ARM Cortex-M3/M4*** — *the* book for this project.
- ARM v7-M Architecture Reference Manual (for settling arguments).
- Miro Samek's course lessons on interrupts/OS basics.

---

## Project 4 — Embedded Software Engineering (`embedded-software-eng/`)

**Theme:** From "code that works" to "code you'd ship." Drivers as products, testing, portability, tooling. **Board:** Nucleo + BME280 + SSD1306 OLED.

**Duration:** ~4 weeks

### Stages

1. **Layered architecture & a real HAL**
   - Design a small, clean HAL interface (yours — informed by, not copied from, ST's) with a strict layer diagram: app / middleware / HAL / registers. Port rule: app layer must compile for both the STM32 target **and** the host PC (mocked HAL).
2. **Real peripheral drivers against real datasheets**
   - **I2C driver** (register-level or LL) → **BME280 driver** on top: read calibration data, apply the (gnarly) compensation formulas, sample temp/humidity/pressure.
   - **SPI driver** → SSD1306 OLED (or I2C variant): framebuffer, fonts, simple graphics.
   - Debug at least one real bus problem with the logic analyzer (there will be one).
3. **Unit testing & CI for embedded**
   - Ceedling/Unity + CMock: test the BME280 compensation math and app logic on the host with a mocked I2C layer; GitHub Actions pipeline that cross-compiles the target build and runs host tests on every push.
   - `clang-format` + static analysis (`cppcheck` or clang-tidy) wired into CI.
4. **Robust firmware patterns**
   - Event-driven state machine framework for the app layer; assertion/error-handling strategy (`ASSERT` → fault report from P3); watchdog usage; non-volatile config storage in a flash page (wear-aware); structured logging over UART with levels.
5. **Ship it**
   - Combine into a polished "sensor dashboard" device: BME280 → OLED + UART logs, buttons to switch views, config persisted to flash, versioned releases with CHANGELOG.

### Definition of Done
- App code compiles and its logic tests pass **on the host PC** with zero hardware — proving the layering is real.
- CI badge green: cross-compile + host tests + lint on every push.
- BME280 + OLED dashboard device runs for days without a glitch (watchdog-supervised).

### Deliverables
Reusable driver set (`i2c`, `spi`, `bme280`, `ssd1306`) · HAL interface + host mock · Ceedling test suite + GitHub Actions workflow · state-machine framework · the dashboard firmware · write-up on architecture decisions.

### Key Resources
- *Making Embedded Systems* (Elecia White) — the spine of this project.
- *Test-Driven Development for Embedded C* (James Grenning).
- BME280 & SSD1306 datasheets; Memfault's Interrupt blog (best practices articles).

---

## Project 5 — Real-Time Systems & RTOS (`embedded-rtos/`)

**Theme:** Concurrency and *deadlines*. FreeRTOS in depth, plus enough theory to reason about schedulability. **Board:** Nucleo (Renode simulation as supplement).

**Duration:** ~4 weeks

### Stages

1. **RTOS mechanics**
   - Port FreeRTOS to your project by hand (no CubeMX): clone kernel, pick the port, write `FreeRTOSConfig.h`, hook SysTick/PendSV/SVC — connect this directly to your P3 toy switcher ("FreeRTOS is my switcher, industrial-grade").
   - Tasks, priorities, `vTaskDelayUntil` vs `vTaskDelay`, idle hook, stack sizing + high-water marks.
2. **Inter-task communication & the classic bugs**
   - Queues, semaphores, mutexes (priority inheritance), event groups, task notifications, stream buffers.
   - **Deliberately construct and then fix:** a race condition, a deadlock, and a priority inversion (recreate the Mars Pathfinder bug). Document each with traces.
3. **Real-time design & analysis**
   - ISR-to-task deferral patterns (`FromISR` APIs), measuring latency and jitter with DWT/GPIO+logic analyzer, rate-monotonic scheduling basics and a schedulability check for your task set.
   - Memory strategy: static allocation (`configSUPPORT_STATIC_ALLOCATION`), heap_4 vs heap_5 trade-offs.
4. **Tracing & observability**
   - Runtime stats, trace hooks; visualize task execution timelines (Tracealyzer trial, Percepio, or SEGGER SystemView if you have a J-Link — else trace-to-UART + a Python plotter).
5. **Refactor Project 4's dashboard onto FreeRTOS**
   - Sensor task, display task, logger task, command-console task, watchdog "sanity" task checking in on all others. Compare architecture before/after.

### Definition of Done
- FreeRTOS brought up from source by hand, no generated code.
- The three classic concurrency bugs demonstrated *and* fixed, with written explanations.
- Multi-task dashboard runs with measured worst-case sensor-to-display latency and a task-timing diagram in the docs.

### Deliverables
FreeRTOS port + config · concurrency-bugs lab (`bugs/race`, `bugs/deadlock`, `bugs/inversion`) · task-timing measurement tooling · RTOS-based dashboard · write-up incl. schedulability analysis.

### Key Resources
- **FreeRTOS official book** (*Mastering the FreeRTOS Real Time Kernel*, free PDF) — read cover to cover.
- Miro Samek's course (RTOS lessons) — he *builds* an RTOS on Cortex-M, perfect after P3.
- *Real-Time Concepts for Embedded Systems* (Li & Yao) for theory.

---

## Project 6 — IoT & Connectivity (`embedded-iot/`)

**Theme:** Putting a device on the network *securely*. **Board:** ESP32 DevKitC with ESP-IDF (which is FreeRTOS-based — P5 knowledge transfers directly).

**Duration:** ~4 weeks

### Stages

1. **ESP-IDF bring-up**
   - Install ESP-IDF, understand its FreeRTOS flavor, build/flash/monitor loop; Wi-Fi station connect with reconnect logic; NVS for credentials; SNTP time sync.
2. **Protocols**
   - **MQTT** properly: QoS levels, retained messages, LWT, topic design; local Mosquitto broker first, then a cloud broker (HiveMQ Cloud / EMQX free tier).
   - HTTP client (REST call to a public API) + a small on-device HTTP server for provisioning; understand when MQTT vs HTTP vs CoAP.
   - BLE basics: advertise sensor readings as a GATT service (stretch).
3. **Security — non-negotiable**
   - TLS for MQTT (server cert validation, then mutual TLS with client certs), secure credential storage, threat-model write-up for your device (spoofing, sniffing, firmware tampering).
4. **Fleet-grade features**
   - **OTA updates** with ESP-IDF's dual-partition scheme + rollback (connect back to the P2 mini-bootloader concept), remote logging, device shadow/desired-vs-reported config pattern, deep-sleep duty cycling with measured battery-life estimate.
5. **Dashboard & pipeline**
   - Telemetry → MQTT → a small backend: Node-RED or a Python bridge → InfluxDB/SQLite → Grafana dashboard. Alert rule (e.g., Telegram message when a threshold is crossed).

### Definition of Done
- ESP32 publishing sensor data over **mutual-TLS MQTT**, surviving router reboots and broker outages gracefully.
- OTA update pushed remotely and verified, with automatic rollback demonstrated on a bad image.
- Live Grafana dashboard + one working alert.

### Deliverables
Provisioning + connectivity manager component · MQTT/TLS telemetry app · OTA pipeline + signed release process · docker-compose for broker/DB/Grafana stack · threat-model doc · write-up.

### Key Resources
- **ESP-IDF Programming Guide** (official docs — excellent).
- HiveMQ's "MQTT Essentials" series.
- OWASP IoT Top 10 (for the threat model).

---

## Project 7 — Integrated Capstone: **Sentinel** (`sentinel/`)

**Theme:** One product-grade device that exercises everything. **Hardware:** STM32 (sensor/control node) + ESP32 (connectivity gateway) talking over UART — a genuinely realistic two-MCU architecture — plus BME280, OLED, battery consideration.

**Duration:** ~5 weeks

### Architecture

```
                    ┌──────────────── Sentinel Node ────────────────┐
  BME280 ──I2C──▶  ┌──────────────┐             ┌───────────────┐  │
  Light  ──ADC──▶  │   STM32F446   │◀── UART ──▶│     ESP32     │──┼── Wi-Fi ──▶ MQTT/TLS ──▶ Grafana
  Button ──EXTI─▶  │  (FreeRTOS)   │  framed     │  (ESP-IDF)    │  │                 │
  OLED   ◀─SPI──   │ sense/store/UI│  protocol   │ net/OTA/shadow│  │                 └──▶ Alerts
                   └──────────────┘             └───────────────┘  │
                    flash log ▲  watchdog ▲       OTA partitions    │
                    └───────────────────────────────────────────────┘
```

### What Each Project Contributes

| Subsystem | Built On |
|-----------|----------|
| Ring-buffered UART link, bit-packed protocol | P1 |
| STM32 boot, linker script awareness, register comfort | P2 |
| Fault handler + crash reporting, low-power sleep between samples, DMA | P3 |
| Driver layer (BME280, OLED), host-tested app logic, CI, flash config, watchdog | P4 |
| FreeRTOS task architecture on both MCUs, measured latencies | P5 |
| Wi-Fi/MQTT/TLS, OTA, dashboard, alerts | P6 |

### Stages

1. **Spec & design week** — requirements doc (functional + timing + power budget), inter-MCU protocol spec (framing, CRC, ack/retry, versioned), task diagrams for both MCUs, repo layout. *Treat this like a real product kickoff.*
2. **Inter-MCU link** — framed, CRC-checked, unit-tested protocol library shared by both firmwares (host-tested per P4 discipline); fault injection (unplug, garbage bytes, resets).
3. **STM32 node firmware** — sensor sampling task, local OLED UI, flash ring-log of readings (survives power loss), sleep between samples, uplink task.
4. **ESP32 gateway firmware** — connectivity manager, device shadow (config flows *down* to STM32: sample rate, thresholds), telemetry uplink with store-and-forward buffering during outages, OTA for the ESP32 **and relayed firmware-update-over-UART for the STM32** (your P2 bootloader knowledge, for real this time).
5. **Cloud & polish** — dashboard, alerts, provisioning flow, enclosure (optional 3D print / project box), 72-hour soak test with logged uptime and current measurements.
6. **Documentation & demo** — architecture doc, "postmortem" write-up, README with photos/GIFs, short demo video. This is the portfolio centerpiece.

### Definition of Done
- 72-hour unattended soak: no lockups (watchdog events logged if any), no data gaps beyond spec, dashboard live throughout.
- Config change made in the cloud reaches the STM32 sensor loop; firmware update delivered end-to-end to *both* MCUs remotely.
- A stranger could rebuild the whole system from the repo docs alone.

---

## Timeline Overview

| Weeks | Project | Milestone Gate |
|-------|---------|----------------|
| 1–2 | P1 Fundamentals | Libraries + Wokwi state machine done |
| 3–6 | P2 Bare-Metal Firmware | Vendor-free UART console on real hardware |
| 7–9 | P3 Cortex-M | PendSV toy switcher works |
| 10–13 | P4 SW Engineering | CI green; dashboard device soaking |
| 14–17 | P5 RTOS | Dashboard refactored onto FreeRTOS; bugs lab done |
| 18–21 | P6 IoT | Mutual-TLS MQTT + OTA + Grafana live |
| 22–26 | P7 Capstone | 72-hour soak passed; demo video |

**Buy hardware by week 2** (Nucleo + basics), **ESP32 + sensors by week 12**.

---

## Working Rules

1. **Every stage ends in something running.** If it doesn't run, the stage isn't done.
2. **Write the README before moving on** — future-you rebuilds from docs, not memory.
3. **Commit early, commit often** — each project is a git repo (or one monorepo); history tells the learning story.
4. **When stuck > 2 hours:** read the reference manual section end-to-end, then check with the debugger/logic analyzer. The answer is almost always in RM0390.
5. **Don't gold-plate early projects.** P1–P3 are about understanding; polish belongs to P4+.
