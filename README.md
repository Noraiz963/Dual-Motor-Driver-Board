# Dual DC Motor Driver (DRV8833)

A dual H-bridge DC motor driver board designed in KiCad, built around the **DRV8833RTY** lets you independently control two DC motors' direction and speed (or drive one bipolar stepper motor) from a microcontroller using simple logic-level PWM inputs.

## Why This Exists

Microcontrollers can't drive motors directly — their GPIO pins can only source a few milliamps, while motors need much higher current and often a separate, higher voltage supply. A motor driver IC sits in between: it takes low-power logic signals from a microcontroller and uses them to switch high-current H-bridge transistors that actually drive the motor, while also handling direction control (forward/reverse) and providing protection features like overcurrent/fault detection.

## Circuit Overview

- **Control input header (J2)**: AIN1/AIN2, BIN1/BIN2 logic inputs for Motor A and Motor B direction/speed control, plus VCC (logic supply) and GND
- **Core driver IC**: DRV8833RTY (U1) — dual H-bridge motor driver handling both motor channels
- **Motor output header (J1)**: AOUT1/AOUT2 and BOUT1/BOUT2 — direct motor connections, plus SLEEP and FAULT broken out for external control/monitoring
- **Motor power rail (VM)**: separate high-current supply input for the motors, decoupled with C1 (10µF) and C2 (0.1µF)
- **Charge pump support**: C3 (2.2µF) on the VCP pin — required external component for the IC's internal gate-drive charge pump
- **Fault indicator**: R1 (4.7kΩ pull-up) + D1 (LED) on the FAULT pin for visual diagnostic feedback
- **Current sense pins**: AISEN, BISEN broken out for current-sensing/current-limiting

## Signal Flow (Concept)

**Control path:**
1. Microcontroller sends PWM/logic signals into AIN1/AIN2 (Motor A) and BIN1/BIN2 (Motor B) via J2.
2. DRV8833 decodes these inputs according to its truth table to determine direction (forward/reverse), brake, or coast state for each motor channel.
3. Internal H-bridge MOSFETs switch accordingly, routing current through the motor windings in the commanded direction.

**Power path:**
- **VCC** — powers only the chip's internal logic circuitry (typically 3.3V–5V from the microcontroller side).
- **VM** — separate, often higher-voltage supply that powers the actual motor current path through the H-bridges. Keeping VM and VCC separate protects the logic circuitry from motor-induced voltage spikes/noise.
- **C1/C2** on VM absorb current spikes from motor start/stop/direction-change events and filter high-frequency switching noise.
- **C3** on VCP supports the internal charge pump, which generates a gate-drive voltage above VM needed to fully switch on the high-side MOSFETs — without it, the H-bridges won't drive correctly.

**Output path:**
- Motor A connects across AOUT1/AOUT2; Motor B connects across BOUT1/BOUT2, both via J1.
- Direction is reversed by swapping which output is high vs. low — no physical rewiring needed, just different logic states on AIN1/AIN2 or BIN1/BIN2.

**Fault/diagnostics:**
- FAULT is pulled high by R1 during normal operation.
- On overcurrent, thermal shutdown, or undervoltage, the DRV8833 pulls FAULT low internally — this toggles D1 (LED) as a visible fault indicator and is also routed out to J1 for the host microcontroller to monitor.

**Sleep mode:**
- SLEEP must be held HIGH for normal operation.
- Pulling SLEEP LOW puts the IC into low-power sleep mode, disabling both motor channels — useful for power saving when motors aren't in use.

**Ground referencing:**
- GND is shared across logic (VCC side), motor power (VM side), and both headers — required for correct switching behavior and accurate fault/current sensing.

## Schematic
![Schematic](https://github.com/Noraiz963/Dual-Motor-Driver-Board/blob/b3afd0a77119023c182e4530f39abe8b292e7e7c/Schematic.png)

## PCB Layout
![PCB Layout](https://github.com/Noraiz963/Dual-Motor-Driver-Board/blob/b3afd0a77119023c182e4530f39abe8b292e7e7c/PCB(MD).png)

## 3D Render
![3D Render](https://github.com/Noraiz963/Dual-Motor-Driver-Board/blob/b3afd0a77119023c182e4530f39abe8b292e7e7c/3d(MD).png)

## Bill of Materials

| Ref | Component | Value |
|-----|-----------|-------|
| U1 | Dual H-Bridge Motor Driver IC | DRV8833RTY |
| J1 | Motor Output Connector (6-pin) | AOUT1/AOUT2, BOUT1/BOUT2, SLEEP, FAULT |
| J2 | Control Input Connector (6-pin) | AIN1/AIN2, BIN1/BIN2, VCC, GND |
| R1 | Resistor (FAULT pull-up) | 4.7kΩ |
| D1 | LED (fault indicator) | — |
| C1 | Capacitor (VM bulk decoupling) | 10µF |
| C2 | Capacitor (VM HF decoupling) | 0.1µF |
| C3 | Capacitor (charge pump, VCP) | 2.2µF |

## Pin Reference

### J2 — Control Input (from Microcontroller)
| Pin | Signal | Description |
|---|---|---|
| 1 | AIN1 | Motor A input 1 (direction/PWM) |
| 2 | AIN2 | Motor A input 2 (direction/PWM) |
| 3 | BIN1 | Motor B input 1 (direction/PWM) |
| 4 | BIN2 | Motor B input 2 (direction/PWM) |
| 5 | GND | Ground reference |
| 6 | VCC | Logic supply (3.3V–5V) |

### J1 — Motor Output
| Pin | Signal | Description |
|---|---|---|
| 1 | SLEEP | Sleep/enable control |
| 2 | AOUT1 | Motor A terminal 1 |
| 3 | AOUT2 | Motor A terminal 2 |
| 4 | BOUT1 | Motor B terminal 1 |
| 5 | BOUT2 | Motor B terminal 2 |
| 6 | FAULT | Fault status output |

> ⚠️ Keep VM within the DRV8833's rated voltage range and ensure adequate current capacity for your motors. Always share GND between the microcontroller and this board.

## Tools Used
- KiCad (schematic capture, PCB layout, 3D visualization)

## Status
Completed schematic design.

## Notes
Built as a learning project to understand H-bridge motor driver operation, charge-pump-based gate driving, and fault/diagnostic circuitry — a reusable module for future robotics or motor-control projects.
