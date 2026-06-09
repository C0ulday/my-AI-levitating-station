# [ ◕‿◕ ] Floating AI Cube

A small AI companion cube that floats under its base, shows a face on a round screen, listens, and talks back.

**Inspirations:** floating globe / moon lamps · desktop AI companions

> Journey: Idea → Prototyping → Hardware → PCB → 3D Design → Maglev station → Code

---

## ✦ The Idea

A **62×62×62 mm** cube that:

- **floats** below an arm-mounted electromagnet (suspension-type maglev)
- runs a **cloud AI pipeline**: voice in → ESP32 → WiFi → speech-to-text → LLM → text-to-speech → speaker
- shows a **face / clock / images** on a round display
- switches **mode when rotated** (IMU) and knows when it's **docked** (Hall)

> **Constraint #1 : weight.** A home-built coil lifts ~100–150 g all-in, so the whole cube is built for the lightest possible mass: thin walls, low infill, minimal components.

![Idea sketch](docs/idea.jpg)

---

## ✦ Architecture

The smart "brain" (wake word → STT → LLM → TTS) lives in the cloud. The ESP32 only captures, displays, and orchestrates.

```
Voice ──► I2S mic ──► ESP32 ──► (WiFi) ──► STT ──► LLM ──► TTS
                        │
                        ├──► round screen (face / clock / image)
                        ├──► I2S amp ──► speaker
                        ├──► MPU6050   (rotation → next mode)
                        ├──► Hall A3144 (docked / removed)
                        └──► Bluetooth ◄── phone audio
```

Maglev station (home-built, *suspension* type — the cube hangs under an arm):

```
Linear Hall ──► Arduino UNO (PID) ──► MOSFET ──► coil (12 V)
      ▲                                            │
      └──────────── field of the N52 magnet ◄──────┘  (in the top of the cube)
```

> ⚠️ **Two different Hall sensors:** a **digital A3144** inside the cube (docked / removed), and a **linear** Hall in the station (position feedback for the PID loop). Don't mix them up.

---

## ✦ Prototyping

Breadboard first to validate the whole "floats + detects rotation/dock + speaks" loop. Build order is deliberate: **cube electronics + software first**, then the **maglev station second** — only once the cube is finished and weighed.

---

## ✦ Hardware

### Cube
| Component | Model | Specs | Notes |
|---|---|---|---|
| MCU | ESP32 NodeMCU (CH340) | classic BT | BT classic needed for phone audio |
| Display | GC9A01 1.28" round IPS SPI | 240×240 | 3.3 V |
| Amp | MAX98357A (I2S) | audio out | **5 V** |
| Speaker | 40 mm | 4 Ω, 3 W | |
| Mic | INMP441 (I2S) | voice in | 3.3 V |
| IMU | MPU6050 (I2C) | rotation → mode | mount perfectly flat |
| Hall | A3144 (digital) | dock / removal | 5 V supply, output pulled to 3.3 V |
| Battery | LiPo 3.7 V 1000 mAh (803040) | energy | |
| Charge | TP4056 USB-C (protected) | recharge | ~0.5 A recommended |
| Boost | MT3608 | 3.7 V → 5 V | **set to ~5.1 V before connecting** |
| Levitated magnet | N52 15×5×2 mm (×1–5) | maglev | stacked for more force, centered in the top |
| Inserts | brass | **M2.5** heat-set | |
| Connectors | JST PH 1.25 | harness | with a loop of slack |

### Maglev station (phase 2)
| Component | Role |
|---|---|
| Coil (enamelled wire + iron/ferrite core) | electromagnet |
| **Linear** Hall (SS49E / A1302 / DRV5055) | position feedback |
| Logic-level MOSFET (IRLZ44N) + flyback diode | power stage |
| Arduino UNO | PID loop |
| 12 V 2–3 A supply | drives the coil |

> Station shape: ~140 mm ballasted base, ~200–220 mm total height, coil support in **PETG** (heat). The base is mains-powered; only the cube is wireless.

> **Power rule (cube):** on the protected TP4056 the load draws from **OUT+** (not BAT); pre-adjust the MT3608 to **~5.1 V** before plugging anything in.

---

## ✦ PCB (KiCad)

This is the project where the **custom PCB** lives. Workflow learned the hard way: don't design the enclosure until the board is real.

```
breadboard ──► freeze schematic (KiCad) ──► route PCB ──► export STEP ──► import into FreeCAD ──► design the cube around the real board
```

---

## ✦ 3D Design

FreeCAD for the cube and the station. Cube priorities: the **levitation magnet dead-centered** at the top, **symmetric mass** for a stable float, a screen window, a speaker grille, a mic port (kept away from the speaker), the Hall sensor facing the base, an access panel for SD/charge, and thin walls to save weight.

![3D Design](docs/3d-design.jpg)

---

## ✦ Sourcing

| Supplier | What |
|---|---|
| Kubii.fr | round display, microSD |
| Gotronic.fr | MAX98357A, INMP441, MPU6050, A3144, TP4056, MT3608, speaker, JST, M2.5 inserts |
| Amazon.fr | LiPo (fastest/safest here), fast restocks |
| AliExpress | cheapest modules + the levitation coil hardware (slow shipping) |

> Order the **levitation coil first** — it sets the weight budget, and everything else is sized around what it can lift.

---

## 💻 Code

C / C++ (Arduino framework on the ESP32). The ESP32 handles capture, display, audio I2S, the IMU/Hall mode state machine, and the WiFi calls to the cloud AI; the station runs a separate **PID** loop on the Arduino UNO.

To do — firmware once the breadboard core is validated.
