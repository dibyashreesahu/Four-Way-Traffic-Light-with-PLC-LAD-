# Four-Way Traffic Light Control System — PLC Ladder Logic

A four-way intersection traffic light controller programmed in **ladder logic (LAD)** for a **Siemens SIMATIC S7-300 (CPU 313C)** using **STEP 7**. The controller drives red/yellow/green signals for four approach roads (East, North, West, South) using a Start/Stop latch and a chain of eight 5-second on-delay timers as the sequencer.

## Overview

- **Platform:** Siemens SIMATIC S7-300, CPU 313C
- **Software:** STEP 7 (SIMATIC Manager), Ladder (LAD) editor
- **Block:** OB1 — Main Program Sweep (Cycle)
- **Networks:** 8 (1 latch + 7 sequencing networks)
- **Cycle time:** 40 seconds (8 phases × 5 seconds)

## How it works

1. **Start/Stop latch (Network 1):** Pressing Start (`I0.0`) sets memory bit `M0.0`, which seals itself in until Stop (`I0.1`) is pressed. Every other network is gated behind `M0.0`.
2. **Timer-chained sequencer (Networks 2–8):** Eight on-delay timers (`T0`–`T7`) are chained so each timer's *done* bit starts the next one. Each timer "owns" a 5-second phase, and the lamps assigned to that phase are wired to switch on for exactly the duration that timer is running.
3. The result is a single rotating green — East → North → West → South → East — with each road getting one green phase and two shared yellow (transition) phases per cycle.

## I/O list

| Address | Signal | Type |
|---|---|---|
| `I0.0` | Start pushbutton | Input |
| `I0.1` | Stop pushbutton | Input |
| `M0.0` | System-running flag (latch) | Memory |
| `Q0.0` | East – Green | Output |
| `Q0.1` | North – Red | Output |
| `Q0.2` | West – Red | Output |
| `Q0.3` | South – Red | Output |
| `Q0.4` | East – Yellow | Output |
| `Q0.5` | North – Yellow | Output |
| `Q0.6` | North – Green | Output |
| `Q0.7` | East – Red | Output |
| `Q1.0` | West – Yellow | Output |
| `Q1.1` | West – Green | Output |
| `Q1.2` | South – Yellow | Output |
| `Q1.3` | South – Green | Output |
| `T0`–`T7` | 5-second on-delay timers (sequencer) | Timer |

## Phase sequence (40-second cycle)

| Phase | Timer | East | North | West | South |
|---|---|---|---|---|---|
| 1 | T0 | 🟢 Green | 🔴 Red | 🔴 Red | 🔴 Red |
| 2 | T1 | 🟡 Yellow | 🟡 Yellow | 🔴 Red | 🔴 Red |
| 3 | T2 | 🔴 Red | 🟢 Green | 🔴 Red | 🔴 Red |
| 4 | T3 | 🔴 Red | 🟡 Yellow | 🟡 Yellow | 🔴 Red |
| 5 | T4 | 🔴 Red | 🔴 Red | 🟢 Green | 🔴 Red |
| 6 | T5 | 🔴 Red | 🔴 Red | 🟡 Yellow | 🟡 Yellow |
| 7 | T6 | 🔴 Red | 🔴 Red | 🔴 Red | 🟢 Green |
| 8 | T7 | 🟡 Yellow | 🔴 Red | 🔴 Red | 🟡 Yellow |

## Network summary

| Network | Fires when | Starts | Outputs driven |
|---|---|---|---|
| 1 | `I0.0` or `M0.0`, and NOT `I0.1` | — | `M0.0` (latch) |
| 2 | `M0.0` AND `T7` done | `T0` | East Green; North/West/South Red |
| 3 | `M0.0` AND `T0` done | `T1` | East Yellow; North Yellow |
| 4 | `M0.0` AND `T1` done | `T2` | North Green; East Red |
| 5 | `M0.0` AND `T2` done | `T3` | West Yellow; North Yellow (2nd) |
| 6 | `M0.0` AND `T3` done | `T4` | West Green |
| 7 | `M0.0` AND `T4`/`T5` done | `T5` | South Yellow; West Yellow (2nd); South Green |
| 8 | `M0.0` AND `T5`/`T6` done | `T6`, `T7` | South Yellow (2nd); East Yellow (2nd) |

## Repository structure

```
four-way-traffic-light-plc/
├── README.md
├── docs/
│   └── Four_Way_Traffic_Light_PLC_Project_Report.docx
├── program/
│   └── FOUR_WAY.pdf              # Ladder logic export (OB1)
└── images/
    └── phase-timing-chart.png    # optional: screenshots/diagrams
```

## Getting started

1. Open **SIMATIC Manager** (STEP 7) and create or open the project.
2. Import/open the **OB1** ladder logic program.
3. Download the program to a real S7-300 CPU 313C, or simulate it with **PLCSIM**.
4. Wire/simulate `I0.0` (Start) and `I0.1` (Stop) and observe `Q0.0`–`Q1.3` cycling through the phase sequence above.

## Notes and possible extensions

- This design uses a **single rotating green** rather than paired opposite roads (North-South / East-West), chosen to demonstrate timer-chained sequencing with the fewest timers.
- Possible extensions: pedestrian crossing signals, sensor/traffic-density-based timing, paired-phase logic, SCADA/HMI visualization, flashing-amber fallback mode.
