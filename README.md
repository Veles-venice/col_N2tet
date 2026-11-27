# Nand2Tetris — From NAND Gates to a Working Computer

Coursework and capstone project for **Nand to Tetris – I (ETCCNT301)**, B.Tech CSE, K. R. Mangalam University.

Starting from a single `Nand` primitive, this repo builds up the full **Hack computer**: logic gates, an arithmetic unit, memory, a CPU, and finally a working computer. It also includes a **two-pass Hack assembler written in Python** and Hack assembly programs that run on the finished machine.

```
NAND ─▶ Gates ─▶ Adders / ALU ─▶ Registers / RAM ─▶ CPU + Memory ─▶ Computer
                                                                      ▲
                          .asm ─▶ assembler.py ─▶ .hack ──────────────┘
```

---

## Repository structure

```
col_N2tet/
├── LAB-1/                  Elementary logic gates
│   ├── NOT Gate            Not  (from Nand)
│   ├── AND gate            And
│   ├── OR                  Or
│   ├── XOR gate            Xor
│   ├── Mux                 Mux
│   └── DMux                DMux
├── LAB-2/                  Boolean arithmetic
│   ├── Half Adder          HalfAdder
│   ├── Full Adder          FullAdder
│   ├── add16               Add16  (16-bit ripple-carry adder)
│   └── Hack ALU            ALU
├── LAB-3/                  Sequential logic / memory
│   ├── BIT                 Bit  (1-bit register, built on DFF)
│   ├── Register            Register  (16-bit)
│   ├── RAM8                RAM8
│   ├── RAM 64              RAM64
│   └── RAM512              RAM512
├── LAB-4/                  Computer architecture
│   ├── CPU.hdl
│   ├── MEMORY.hdl
│   └── Computer.hdl
├── Capstone/
│   ├── PART A/             Final CPU.hdl, Memory.hdl, Computer.hdl
│   ├── PART B/             Test programs (.asm) and their machine code (.hack)
│   │                       Add, Max, Rect, Pong
│   └── PART C/             assembler.py + Box.asm / Box.hack
├── NAND2TETRIS_PDF.pdf             Lab record: expressions, logic diagrams,
│                                   truth tables and HDL listings
└── Nand2Tetris_Capstone_Report.pdf Capstone report with test evidence
```

---

## What's inside

### Labs 1–3: chips

| Lab | Chips | Notes |
|-----|-------|-------|
| 1 | `Not`, `And`, `Or`, `Xor`, `Mux`, `DMux` | Every gate is derived from `Nand`. |
| 2 | `HalfAdder`, `FullAdder`, `Add16`, `ALU` | `Add16` chains 16 full adders. The ALU supports the six Hack control bits (`zx nx zy ny f no`) and the `zr`/`ng` status flags. |
| 3 | `Bit`, `Register`, `RAM8`, `RAM64`, `RAM512` | Each RAM is built from eight of the next-smaller unit, using `DMux8Way` to route `load` and `Mux8Way16` to select the output. |

### Lab 4 and Capstone Part A: the Hack computer

- **CPU**: decodes A- and C-instructions, drives the A/D registers and the ALU, and handles jumps through the program counter.
- **Memory**: a 32K address space mapped to `RAM16K` (0–16383), `Screen` (16384–24575) and `Keyboard` (24576).
- **Computer**: `ROM32K`, `CPU` and `Memory` wired together.

The capstone report records these chips passing the official test scripts in the Hardware Simulator: `CPU.tst`, `Memory.tst`, `ComputerAdd.tst`, `ComputerMax.tst` and `ComputerRect.tst`.

### Capstone: Python Hack assembler

[`Capstone/PART C/assembler.py`](Capstone/PART%20C/assembler.py) is a dependency-free, two-pass assembler:

1. **Pass 1** removes comments and blank lines and records each `(LABEL)` against its ROM address.
2. **Pass 2** translates A-instructions (`@value`, `@symbol`) and C-instructions (`dest=comp;jump`) into 16-bit binary. New variables are assigned RAM addresses starting at 16.

It supports all predefined symbols (`R0`–`R15`, `SP`, `LCL`, `ARG`, `THIS`, `THAT`, `SCREEN`, `KBD`), commutative forms such as `A+D` and `M|D`, and both `MD` and `DM` destinations.

**Extra feature:** it warns, with the line number, when a label is declared more than once and keeps the first definition. `PART B/Add.asm` declares `(LOOP)` twice on purpose to show this.

### Capstone: assembly programs

| Program | What it does |
|---------|--------------|
| `Add.asm` | Computes `R0 = 2 + 3`. |
| `Max.asm` | Computes `R2 = max(R0, R1)`. |
| `Rect.asm` | Draws a rectangle 16 pixels wide and `RAM[0]` rows tall at the top-left of the screen. |
| `Pong.asm` | A simplified Pong paddle loop, used to test the assembler. |
| `Box.asm` | Draws a filled black box while any key is pressed and clears it when the key is released. It polls `KBD` and uses nested row/column loops to write to screen memory. |

---

## Getting started

### Requirements

- [Nand2Tetris Software Suite](https://www.nand2tetris.org/software) (Hardware Simulator and CPU Emulator), or the online [Nand2Tetris IDE](https://nand2tetris.github.io/web-ide)
- Python 3.9 or later (for the assembler)

### Running the assembler

```bash
cd "Capstone/PART C"
python3 assembler.py Box.asm
# Success: Box.asm -> Box.hack (53 instructions)
```

The output `.hack` file is written next to the input file. To assemble the Part B programs:

```bash
python3 assembler.py "../PART B/Max.asm"
```

### Running a program on the Hack computer

1. Open the **CPU Emulator**.
2. Load a `.hack` (or `.asm`) file into ROM.
3. For `Box`, enable keyboard input, press **Run**, then hold any key to make the box appear.

### Testing the HDL chips

1. Lab files are saved without the `.hdl` extension and some use descriptive names (for example `AND gate`). Copy each one into the matching project folder of the official suite with its chip name, for example `And.hdl` or `ALU.hdl`.
2. Open the matching `.tst` script in the **Hardware Simulator** and run it. A passing chip ends with *"Comparison ended successfully"*.

---

## Known issues

- **`Box.hack` is out of date.** Reassembling `Box.asm` gives a different start address on line 11 (`@16800`) from the committed file. Run the assembler again to refresh it.
- **No validation of invalid instructions.** An unknown `comp`, `dest` or `jump` currently raises a Python `KeyError` instead of a friendly error.
- **The report doesn't match the code in a few places.** `Nand2Tetris_Capstone_Report.pdf` describes an earlier version of the project, with a different personal *K* value, a different box size and a longer `Pong.asm`. The source files in the repo are the current version.

---

## Documentation

- **[NAND2TETRIS_PDF.pdf](NAND2TETRIS_PDF.pdf)**: lab record with the Boolean expression, logic diagram, truth table and HDL listing for each chip.
- **[Nand2Tetris_Capstone_Report.pdf](Nand2Tetris_Capstone_Report.pdf)**: capstone report with simulator screenshots, the assembler write-up and the design of the box program.

## Acknowledgements

Based on *The Elements of Computing Systems* by Noam Nisan and Shimon Schocken (MIT Press) and the [Nand2Tetris](https://www.nand2tetris.org) course materials. Course faculty: Ms. Nainsi.# Nand2Tetris — From NAND Gates to a Working Computer

Coursework and capstone project for **Nand to Tetris – I (ETCCNT301)**, B.Tech CSE, K. R. Mangalam University.

Starting from a single `Nand` primitive, this repo builds up the full **Hack computer**: logic gates, an arithmetic unit, memory, a CPU, and finally a working computer. It also includes a **two-pass Hack assembler written in Python** and Hack assembly programs that run on the finished machine.

```
NAND ─▶ Gates ─▶ Adders / ALU ─▶ Registers / RAM ─▶ CPU + Memory ─▶ Computer
                                                                      ▲
                          .asm ─▶ assembler.py ─▶ .hack ──────────────┘
```

---

## Repository structure

```
col_N2tet/
├── LAB-1/                  Elementary logic gates
│   ├── NOT Gate            Not  (from Nand)
│   ├── AND gate            And
│   ├── OR                  Or
│   ├── XOR gate            Xor
│   ├── Mux                 Mux
│   └── DMux                DMux
├── LAB-2/                  Boolean arithmetic
│   ├── Half Adder          HalfAdder
│   ├── Full Adder          FullAdder
│   ├── add16               Add16  (16-bit ripple-carry adder)
│   └── Hack ALU            ALU
├── LAB-3/                  Sequential logic / memory
│   ├── BIT                 Bit  (1-bit register, built on DFF)
│   ├── Register            Register  (16-bit)
│   ├── RAM8                RAM8
│   ├── RAM 64              RAM64
│   └── RAM512              RAM512
├── LAB-4/                  Computer architecture
│   ├── CPU.hdl
│   ├── MEMORY.hdl
│   └── Computer.hdl
├── Capstone/
│   ├── PART A/             Final CPU.hdl, Memory.hdl, Computer.hdl
│   ├── PART B/             Test programs (.asm) and their machine code (.hack)
│   │                       Add, Max, Rect, Pong
│   └── PART C/             assembler.py + Box.asm / Box.hack
├── NAND2TETRIS_PDF.pdf             Lab record: expressions, logic diagrams,
│                                   truth tables and HDL listings
└── Nand2Tetris_Capstone_Report.pdf Capstone report with test evidence
```

---

## What's inside

### Labs 1–3: chips

| Lab | Chips | Notes |
|-----|-------|-------|
| 1 | `Not`, `And`, `Or`, `Xor`, `Mux`, `DMux` | Every gate is derived from `Nand`. |
| 2 | `HalfAdder`, `FullAdder`, `Add16`, `ALU` | `Add16` chains 16 full adders. The ALU supports the six Hack control bits (`zx nx zy ny f no`) and the `zr`/`ng` status flags. |
| 3 | `Bit`, `Register`, `RAM8`, `RAM64`, `RAM512` | Each RAM is built from eight of the next-smaller unit, using `DMux8Way` to route `load` and `Mux8Way16` to select the output. |

### Lab 4 and Capstone Part A: the Hack computer

- **CPU**: decodes A- and C-instructions, drives the A/D registers and the ALU, and handles jumps through the program counter.
- **Memory**: a 32K address space mapped to `RAM16K` (0–16383), `Screen` (16384–24575) and `Keyboard` (24576).
- **Computer**: `ROM32K`, `CPU` and `Memory` wired together.

The capstone report records these chips passing the official test scripts in the Hardware Simulator: `CPU.tst`, `Memory.tst`, `ComputerAdd.tst`, `ComputerMax.tst` and `ComputerRect.tst`.

### Capstone: Python Hack assembler

[`Capstone/PART C/assembler.py`](Capstone/PART%20C/assembler.py) is a dependency-free, two-pass assembler:

1. **Pass 1** removes comments and blank lines and records each `(LABEL)` against its ROM address.
2. **Pass 2** translates A-instructions (`@value`, `@symbol`) and C-instructions (`dest=comp;jump`) into 16-bit binary. New variables are assigned RAM addresses starting at 16.

It supports all predefined symbols (`R0`–`R15`, `SP`, `LCL`, `ARG`, `THIS`, `THAT`, `SCREEN`, `KBD`), commutative forms such as `A+D` and `M|D`, and both `MD` and `DM` destinations.

**Extra feature:** it warns, with the line number, when a label is declared more than once and keeps the first definition. `PART B/Add.asm` declares `(LOOP)` twice on purpose to show this.

### Capstone: assembly programs

| Program | What it does |
|---------|--------------|
| `Add.asm` | Computes `R0 = 2 + 3`. |
| `Max.asm` | Computes `R2 = max(R0, R1)`. |
| `Rect.asm` | Draws a rectangle 16 pixels wide and `RAM[0]` rows tall at the top-left of the screen. |
| `Pong.asm` | A simplified Pong paddle loop, used to test the assembler. |
| `Box.asm` | Draws a filled black box while any key is pressed and clears it when the key is released. It polls `KBD` and uses nested row/column loops to write to screen memory. |

---

## Getting started

### Requirements

- [Nand2Tetris Software Suite](https://www.nand2tetris.org/software) (Hardware Simulator and CPU Emulator), or the online [Nand2Tetris IDE](https://nand2tetris.github.io/web-ide)
- Python 3.9 or later (for the assembler)

### Running the assembler

```bash
cd "Capstone/PART C"
python3 assembler.py Box.asm
# Success: Box.asm -> Box.hack (53 instructions)
```

The output `.hack` file is written next to the input file. To assemble the Part B programs:

```bash
python3 assembler.py "../PART B/Max.asm"
```

### Running a program on the Hack computer

1. Open the **CPU Emulator**.
2. Load a `.hack` (or `.asm`) file into ROM.
3. For `Box`, enable keyboard input, press **Run**, then hold any key to make the box appear.

### Testing the HDL chips

1. Lab files are saved without the `.hdl` extension and some use descriptive names (for example `AND gate`). Copy each one into the matching project folder of the official suite with its chip name, for example `And.hdl` or `ALU.hdl`.
2. Open the matching `.tst` script in the **Hardware Simulator** and run it. A passing chip ends with *"Comparison ended successfully"*.

---

## Known issues

- **`Box.hack` is out of date.** Reassembling `Box.asm` gives a different start address on line 11 (`@16800`) from the committed file. Run the assembler again to refresh it.
- **No validation of invalid instructions.** An unknown `comp`, `dest` or `jump` currently raises a Python `KeyError` instead of a friendly error.
- **The report doesn't match the code in a few places.** `Nand2Tetris_Capstone_Report.pdf` describes an earlier version of the project, with a different personal *K* value, a different box size and a longer `Pong.asm`. The source files in the repo are the current version.

---

## Documentation

- **[NAND2TETRIS_PDF.pdf](NAND2TETRIS_PDF.pdf)**: lab record with the Boolean expression, logic diagram, truth table and HDL listing for each chip.
- **[Nand2Tetris_Capstone_Report.pdf](Nand2Tetris_Capstone_Report.pdf)**: capstone report with simulator screenshots, the assembler write-up and the design of the box program.

## Acknowledgements

Based on *The Elements of Computing Systems* by Noam Nisan and Shimon Schocken (MIT Press) and the [Nand2Tetris](https://www.nand2tetris.org) course materials. Course faculty: Ms. Nainsi.
