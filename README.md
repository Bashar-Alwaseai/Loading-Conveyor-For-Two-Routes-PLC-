# 🏭 Loading Conveyor Control for Two Routes — PLC (Siemens LOGO!)

An industrial **truck-loading conveyor system** controlled by a **Siemens LOGO! PLC**. Two feeding conveyor lines carry material toward a shared conveyor, and the PLC decides **which line runs**, in **Manual** or **Automatic** mode, with **safety interlocks** so the lines never conflict.

![PLC](https://img.shields.io/badge/PLC-Siemens%20LOGO!%200BA8-009999)
![Software](https://img.shields.io/badge/software-LOGO!%20Soft%20Comfort-blue)
![Language](https://img.shields.io/badge/language-Ladder%20(LAD%2FKOP)-orange)
![Simulation](https://img.shields.io/badge/simulation-LOGO!%20Soft%20Comfort-success)

---

## 📌 Overview (the simple version)

Picture a loading yard with **two trucks**. Each truck is loaded by its own conveyor line, and both lines merge into one conveyor that carries the material onward.

![Concept: two lines merging into one conveyor](images/concept_trucks_1.jpg)

The PLC program does three jobs:

1. **Starts and stops the motors in order** — each conveyor motor has Start/Stop buttons and a circuit-breaker (CB) input.
2. **Chooses the route** — in automatic mode only **one line can run at a time** (Line 1 *or* Line 2), so material from both lines never collides at the merge point.
3. **Keeps everything safe** — a tripped circuit breaker or a pressed Stop button immediately drops the motor.

A **mode selector** (`Auto/Manual`) switches between operating the conveyors by hand and letting the automatic sequence take over.

![Concept: material moving to the second line](images/concept_trucks_2.jpg)

This is the kind of system used in real loading plants (for example, loading bags onto trucks):

![Real loading plant](images/real_loading_plant.jpg)

---

## ✨ Features

| Feature | Description |
|---|---|
| 🔀 **Manual / Auto mode** | One selector input (`I1`) chooses between the two control sections |
| 🔒 **Interlocking** | In auto mode, Line 1 and Line 2 are mutually exclusive (each blocks the other) |
| 🧲 **Latching (seal-in) start/stop** | Start pushbutton + holding contact keeps a motor running until Stop |
| 🛑 **Safety chain** | Circuit-breaker (`CB_x`) and Stop (`Stop_x`) contacts are in series with every motor |
| ↔️ **Diverter directions** | Left / Right and Left_Middle / Right_Middle outputs control the material direction; the middle pair is interlocked so both can't be on together |
| 🖥️ **Simulated & verified** | Tested in LOGO! Soft Comfort simulation mode (see screenshots below) |

---

## 🧰 Hardware / Software

| Item | Details |
|---|---|
| PLC | Siemens **LOGO!** (device profile **0BA8**, standard) |
| Programming tool | **LOGO! Soft Comfort** — Ladder diagram (KOP/LAD) |
| Program file | `Load Line Control.lld` |
| 3D plant layout | Prepared in **Factory I/O** (see below) |

![Factory I/O layout](images/factory_io_layout.jpg)

---

## 🔌 Inputs & memory flags (from the program)

### Inputs

| Input | Name | Function |
|:---:|---|---|
| `I1` | **Auto/Manual** | Mode selector (manual section when closed, auto section when open) |
| `I2` | **CB_1** | Circuit breaker, line 1 (break contact) |
| `I3` | **Stop_1** | Stop, line 1 (break contact) |
| `I4` | **Start_1** | Start, line 1 |
| `I6` | **CB_2** | Circuit breaker, line 2 (break contact) |
| `I7` | **Stop_2** | Stop, line 2 (break contact) |
| `I8` | **Run_2** | Start, line 2 |
| `I10` | **CB_3** | Circuit breaker, conveyor 3 (break contact) |
| `I11` | **Stop_3** | Stop, conveyor 3 (break contact) |
| `I12` | **Run_3** | Start, conveyor 3 |
| `I14` | **Run_4** | Start, conveyor 4 |
| `I16` | **CB_4** | Circuit breaker, conveyor 4 |
| `I17` | **Stop_4** | Stop, conveyor 4 |
| `I18` | **Run_5** | Start, conveyor 5 |
| `I15` | — | Extra enable contact in the auto section (unnamed in the file) |

### Internal flags and outputs

| Flag | Name | Role |
|:---:|---|---|
| `M1` | **Left** | Manual: motor/direction "Left" |
| `M2` | **Right** | Manual: motor/direction "Right" |
| `M3` | **Left_Middle** | Manual: middle conveyor, left direction |
| `M4` | **Right_Middle** | Manual: middle conveyor, right direction (interlocks `M3`) |
| `M11` | **Feedback_1** | Auto: **Line 1 active** |
| `M12` | — | Auto: **Line 2 active** (interlocked with `M11`) |
| `M13` / `M14` | — | Auto: follow-on conveyors of Line 1 / Line 2 |
| `Q1`–`Q5` | — | Physical outputs that drive the conveyor motors |

> The names above come from the labels saved in `Load Line Control.lld`. Update the table if you rename any I/O.

---

## 🧠 How it works (the deeper version)

### Manual section (`I1` closed)
Each motor is a classic **start/stop seal-in rung**:

```
I1 ──[ I2/ ]──[ I3/ ]──┬──[ I4 ]──┬──( M1 )     ← Start_1 latches M1 ("Left")
                       └──[ M1 ]──┘
```
- `I2` (CB) and `I3` (Stop) are **break contacts** in series, so tripping the breaker or pressing Stop drops the motor.
- The holding contact `M1` keeps the output on after Start is released.
- `M2` ("Right") uses the same structure with `I6`, `I7`, `I8`.
- `M3` ("Left_Middle") uses `I10`, `I11`, `I12` and adds a **break contact of `M4`** — the interlock that stops both directions from running at once.

![Manual section — ladder diagram](images/ladder_manual.png)

### Automatic section (`I1` open)
The same seal-in pattern is used, with an added **mutual interlock** between the two lines:

```
I1/ ─ I15/ ─┬─[ I2/ ]─[ I3/ ]─┬[ I4 ]┬─[ M12/ ]──( M11 )   ← Line 1, blocked when Line 2 runs
            │                 └[ M11 ┘
            └─[ I6/ ]─[ I7/ ]─┬[ I8 ]┬─[ M11/ ]──( M12 )   ← Line 2, blocked when Line 1 runs
                              └[ M12 ┘
```
- `M11` and `M12` each contain a **break contact of the other**, so only one line can be active at a time.
- `M13` is enabled by `M11` (Line 1) and `M14` by `M12` (Line 2): the conveyors that follow a line start only when that line is selected.

![Automatic section — ladder diagram](images/ladder_auto.png)

### Simulation results
The program was tested in LOGO! Soft Comfort's simulation mode (the I/O bar at the bottom shows inputs and outputs live).

| Scenario | Result seen in simulation |
|---|---|
| **Line 1 start** (`I4` pressed, Line 2 stopped) | `M11`, `M13` → outputs **`Q1`, `Q3`** on |
| **Line 1 stopped, Line 2 start** (`I3`, `I8` pressed) | `M12`, `M14` → outputs **`Q2`, `Q4`** on, Line 1 off |

![Simulation — Line 1 running](images/simulation_line1.png)

![Simulation — Line 2 running](images/simulation_line2.png)

---

## 📁 Repository structure

```
.
├── README.md
├── Load Line Control.lld        # LOGO! Soft Comfort ladder program
└── images/
    ├── ladder_manual.png        # manual section
    ├── ladder_auto.png          # automatic section
    ├── simulation_line1.png     # simulation: Line 1 running
    ├── simulation_line2.png     # simulation: Line 2 running
    ├── concept_trucks_1.jpg     # concept illustration
    ├── concept_trucks_2.jpg
    ├── real_loading_plant.jpg   # real-world reference
    └── factory_io_layout.jpg    # Factory I/O plant layout
```

---

## 🚀 Getting started

1. Install **LOGO! Soft Comfort** (Siemens).
2. Open `Load Line Control.lld`.
3. Press **Simulation** and toggle the inputs (`I1`, `I4`, `I8`, …) in the bottom bar.
   - `I1` = 0 → automatic section · `I1` = 1 → manual section.
   - Remember that CB/Stop inputs are **break contacts** (the rung passes while the input is `0`).
4. To run on a real device, connect the LOGO! (0BA8) and use **Tools → Transfer → PC → LOGO!**, wiring `Q1`–`Q5` to the motor contactors.

---

## 🔭 Possible improvements
- Add an **emergency stop** that resets every flag at once.
- Add **overload/timeout** protection and an alarm lamp.
- Add **on-delay timers** for sequential motor start-up to reduce inrush current.
- Link the program to the **Factory I/O** scene for a full 3D simulation.

---

## 👤 Author

*Add your name, university, and supervisor here.*

---

## 📄 License

Add a license of your choice (e.g. [MIT](https://choosealicense.com/licenses/mit/)) by creating a `LICENSE` file in the repository root.
