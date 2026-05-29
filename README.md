# esphome-nidec-neptune-pool-controller
ESPHome configuration for the Waveshare Industrial ESP32-S3-Relay-6CH.  Built to integrate with the Nidec Neptune (NPTQ270) Modbus protocol, native to Waterway Power Defender (PD) variable speed pumps. Features robust split-surface intent architecture, adaptive RPM guardrails, safety-locked liquid chlorine dosing injection, and 7-day rolling historical telemetry tracking.

## Built and Tested On:

* **Controller Board:** Waveshare Industrial ESP32-S3-Relay-6CH (utilizing onboard RS-485 transceiver, piezo buzzer, and WS2812 status indicator)
* **Circulation Pump:** Waterway Power Defender 140 (PD140) Variable Speed Pump (powered by the Nidec Neptune NPTQ270 drive engine via Modbus RTU)
* **Dosing Hardware:** Stenner 7.5-Gallon Tank System with an integrated fixed-rate peristaltic pump (driven via isolated onboard Relay 1)

## Overview of Features & Functions

### Core Architecture & Communication

* **Onboard RS485 Implementation:** Leverages the Waveshare 6-Channel dedicated hardware RS-485 infrastructure to establish the `9600` baud Modbus link directly to the pump's data terminal.
* **Direct Register-Level Control:** Eliminates the need for custom external C++ components by managing direct Modbus RTU register mapping using ESPHome's native `modbus_controller` block.
* **Command Throttling:** Built-in hardware-safe communication packet pacing (`200ms` throttle) to prevent overflowing the Nidec microprocessor serial buffer.
* **High-Voltage Expansion Path:** Uses Relay 1 to switch the line-voltage AC to the Stenner pump, leaving 5 dry-contact relays completely open for future pad expansions (e.g., pool lights, cleaner pumps, or heater loops).

### Water Pump Management

* **Multi-Tier Guardrail Limits:** Interlocking physical hardware limits (`600`–`3450` RPM) with dynamic user-facing operating boundaries and temporary service overrides (e.g., vacuuming or backwashing).
* **Clamped Target Command Surface:** Separates public operator "intent" from the low-level execution command path. The requested RPM is sanitized and clamped by active limits locally before writing to the drive registers.
* **Algorithmic Flow Estimation:** Embedded C++ multi-point linear interpolation array that derives continuous real-time GPM (Gallons Per Minute) estimation mapping across the full RPM torque curve.

### Liquid Chlorine Dosing Subsystem

* **Hardware-Calibrated Volumetric Math:** Uses a default calculation of 1.0 ounce per minute (tailored to standard Stenner outputs) to convert high-level fluid ounce or PPM automated requests into exact relay runtime cycles.
* **Isolated Intent Requesting:** Allows precise volumetric staging in fluid ounces or chemical target metrics in Parts-Per-Million (PPM), with automated chemistry back-calculations based on chemical liquid strength and pool volume.
* **Virtual Tank Reservoir Mirroring:** Tracks consumption dynamically to maintain a highly accurate local software mirror of the physical 7.5-gallon (960 oz) Stenner tank level, surfacing low-threshold diagnostic alerts before the pump runs dry.
* **Asynchronous Safety Gates:** Dosing can only be initialized through an explicit secondary execution command after clearing five safety gates: verified calibration data, valid requests, maintenance lockout overrides off, active Modbus telemetry loop confirmation, and confirmed water movement.
* **Automated Fault Mitigation:** Constant diagnostic loops will instantly trip a fallback shutoff script and abort active dosing schedules if the main pool pump unexpectedly drops communication or loses physical velocity.
* **Chemical Siphoning Safeguards:** Implements strict pre-run and post-run delays to ensure the water line is moving at high velocity before the Stenner pump activates, entirely mitigating the risk of dumping raw liquid chlorine into stagnant PVC plumbing.

### Telemetry Tracking & Hardware UI

* **7-Day Rolling History Matrix:** Shifting arrays that persist and track water circulation runtime, operational energy metrics ($Wh$), overall volume throughput ($gal$), and chlorine consumption rates directly inside local flash storage.
* **Hardware-Level Alarms:** On-board UI utilization including passive piezo buzzer melody routing via LEDC PWM (ideal for close-proximity audio warnings at the physical equipment pad) and WS2812 status LED illumination.

## Screenshots

### ESPHome Default Web View
<img width="909" height="3209" alt="image" src="https://github.com/user-attachments/assets/ffaee8e3-3063-46b0-a4ea-e59d0f9a9fea" />

### My Home Assistant Dashboard (Custom and not included in this repo, but gives an idea of what is possible)
<img width="395" height="748" alt="image" src="https://github.com/user-attachments/assets/eba97ccc-87ec-42ac-a07a-e77923896973" />
<img width="416" height="734" alt="image" src="https://github.com/user-attachments/assets/5cfc8092-c6c2-4d28-ae41-642d42f8d3cc" />
<img width="406" height="810" alt="image" src="https://github.com/user-attachments/assets/3c443b99-c7d7-4ef1-9fd9-cbeb4580e611" />
<img width="406" height="732" alt="image" src="https://github.com/user-attachments/assets/a76670ce-7bfc-4acb-8128-ae67f694ccfe" />

