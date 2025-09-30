# 📘 KKT621RISC Assembler

This is a **Python assembler** for the **KKT621RISC processor**.
It translates assembly source files into Memory Initialization Files (`.mif`) for FPGA simulations.

---

## ✨ Features

- Supports 20 Manipulation instructions:
  `ADD, SUB, ADDC, SUBC, NOT, AND, OR, XOR, SRA, CPY, SWAP, RRC, RRV, RLN, RLZ, ROTR, ROTL, SRL, MUL, DIV`

- Instruction word size: **15 bits**
- Program memory depth: **1024 words**
- Stepwise flow:
  1. Read + syntax check
  2. Split into segments (`.directives`, `.constants`, `.data`, `.code`)
  3. Assemble into binary words
  4. Write `.mif` output with filler zeros

- Generates debug text files:
  - `dir.txt` → assembler directives
  - `const.txt` → constants
  - `data.txt` → data
  - `code.txt` → code

- Only emits constants/data if referenced in the `.code` section.

---

## 📂 Project Structure

kkt621risc/
├── main.py # Entry point
├── isa.py # ISA definitions + encode/decode helpers
├── parser.py # read_file, split_segments, dump_text
├── assembler.py # assemble_code (core translation)
└── writer.py # write_mif


---

## ⚙️ Requirements

- Python 3.11+
- No external dependencies (uses only `sys` and `re`).

---

## ▶️ Usage

### 1. Run assembler
``` 
python main.py
```

It will prompt:

```
Enter source file [default: program.txt]:
```

Press Enter → assembles program.txt

Or type another filename (e.g. all_instr_test.asm)

### 2. Output
Main result: output.mif

Debug: dir.txt, const.txt, data.txt, code.txt

Console log shows step-by-step process.

## 📂 Assembly File Format

Assembly source uses segments:

.directives;
  .equ constOne 0x1;
  .equ constTwo 0x2;
.enddirectives;

.constants;
  .word firstConstWord 0xFFFF;
.endconstants;

.data;
  .word myData 0x123;
.enddata;

.code;
  SUB R2, R2;
  ADDC R2, 0x3;
  NOT R1;
.endcode;

## 🖥️ Example
Input (program.txt)
.code;
  SUB R2, R2;
  SUB R3, R3;
  ADDC R2, 0x3;
  NOT R1;
.endcode;

Output (output.mif)
--Program Memory Initialization File
--Created by kkt621RISC assembler
WIDTH = 15;
DEPTH = 1024;
ADDRESS_RADIX = HEX;
DATA_RADIX = BIN;

CONTENT BEGIN

--A> : <OpCode><-Ri-><-Rj->
0000 : 001100001000010; % SUB R2, R2; %
0001 : 001100001100011; % SUB R3, R3; %
0002 : 001110001000011; % ADDC R2, 0x3; %
0003 : 010010000100000; % NOT R1; %
[ 0004 .. 3FF ] : 000000000000000; % Fill remaining 0%
END;

## 🔁 Flow Diagram

Here’s how the assembler works step by step:

                 ┌───────────────────────┐
                 │ Start Assembler (main)│
                 └─────────────┬─────────┘
                               │
                               ▼
              ┌────────────────────────────────┐
              │ Step 1: Read Source File       │
              │ - Open input .asm file         │
              │ - Load all lines into memory   │
              └────────────────┬───────────────┘
                               │
                               ▼
              ┌────────────────────────────────┐
              │ Step 2: Split Segments         │
              │ - Separate lines into:         │
              │   .directives / .constants /   │
              │   .data / .code                │
              │ - Save each to debug .txt files│
              └────────────────┬───────────────┘
                               │
                               ▼
              ┌────────────────────────────────┐
              │ Step 3: Assemble Code          │
              │ - Parse directives (.equ)      │
              │   → add symbols to table       │
              │ - Parse constants/data (.word) │
              │   → store but emit only if used│
              │ - Assemble .code instructions  │
              │   → convert to 15-bit binary   │
              │ - Track labels & operands      │
              └────────────────┬───────────────┘
                               │
                               ▼
              ┌────────────────────────────────┐
              │ Step 4: Write Output .mif      │
              │ - Write header (WIDTH, DEPTH)  │
              │ - Dump machine words + comments│
              │ - Fill rest of memory with 0   │
              │ - End with END;                │
              └────────────────┬───────────────┘
                               │
                               ▼
                 ┌────────────────────────────┐
                 │  Assembly Successful!      │
                 │  Files generated:          │
                 │   - output.mif             │
                 │   - dir.txt, const.txt     │
                 │   - data.txt, code.txt     │
                 └────────────────────────────┘

## 📌 Notes
Constants/data only appear in .mif if they are referenced.
Labels can be defined in .code and used as operands.
All registers are written as R0…R31.

## 📜 License
Free to use for academic and research purposes.
