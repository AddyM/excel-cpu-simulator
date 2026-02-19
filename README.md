# 8-Bit CPU Simulator in Pure Excel Formulas

> A fully functional 8-bit CPU simulated entirely in Microsoft Excel / Google Sheets.  
> Zero VBA. Zero macros. Zero code. Just formulas.

---

## What Is This?

A working 8-bit CPU built entirely out of spreadsheet formulas. Every register, every flag, every clock cycle is computed by a chain of `IFS`, `SWITCH`, `INDEX`, `BITAND`, and `MIN/MAX` formulas. Change a value in RAM and watch all 64 clock cycles recompute instantly.

---

## Architecture

| Component | Implementation |
|-----------|---------------|
| RAM | 256-byte memory grid (16×16 cells), fully editable |
| Registers | A (Accumulator), B (General Purpose), SP (Stack Pointer), PC (Program Counter) |
| Flags | Z (Zero), C (Carry), N (Negative) |
| ALU | ADD, SUB, AND, OR, XOR, INC, DEC, NOT — all pure formulas |
| Control Unit | Fetch → Decode → Execute via IFS formula chain |
| Execution Trace | 64-step table, one row per clock cycle |

---

## Instruction Set (15 Instructions)

| Opcode | Mnemonic | Operation |
|--------|----------|-----------|
| 0x00 | NOP | No operation |
| 0x01 | LDA #n | A = n |
| 0x02 | LDB #n | B = n |
| 0x03 | STA @a | MEM[a] = A |
| 0x04 | ADD | A = A + B |
| 0x05 | SUB | A = A - B |
| 0x06 | AND | A = A AND B |
| 0x07 | OR | A = A OR B |
| 0x08 | XOR | A = A XOR B |
| 0x09 | INC | A = A + 1 |
| 0x0A | DEC | A = A - 1 |
| 0x0B | JMP @a | PC = a |
| 0x0C | JZ @a | if Z=1: PC = a |
| 0x0D | JNZ @a | if Z=0: PC = a |
| 0xFF | HLT | Halt execution |

---

## How to Open

### Google Sheets (Recommended)
1. Go to [sheets.google.com](https://sheets.google.com)
2. File → Import → Upload the `.xlsx` file
3. Open the **DASHBOARD** tab
4. Change the number in cell **C4** and watch everything update instantly

### Microsoft Excel (Windows)
1. Open the file, click **Enable Editing** if prompted
2. Open the **DASHBOARD** tab
3. Change cell **C4**, press **F9** to recalculate

### Microsoft Excel (Mac)
1. Open the file, click **Enable Editing** if prompted
2. Open the **DASHBOARD** tab
3. Change cell **C4**, press **Cmd =** or Formulas → Calculate Now

---

## Hands-On Walkthrough

### Program 1 — Load a number (simplest possible program)

**Go to the RAM sheet. Change the first row (0x00) to:**

| +0 | +1 | +2 | +3 | +4 | +5 |
|----|----|----|----|----|-----|
| 1 | 42 | 255 | 0 | 0 | 0 |

- `1` = opcode for LDA (load into A)
- `42` = the value to load
- `255` = HLT (stop)

**Go to DASHBOARD and step through:**

- C4 = `0` → Accumulator shows `0x00`. Nothing has executed yet.
- C4 = `1` → Accumulator shows `0x2A`. That is 42 in hex. LDA just fired.
- C4 = `2` → STATUS shows `HALTED`. Program complete.

---

### Program 2 — Add two numbers

**Go to RAM sheet. Change row 0x00 to:**

| +0 | +1 | +2 | +3 | +4 | +5 | +6 | +7 |
|----|----|----|----|----|----|----|-----|
| 1 | 7 | 2 | 5 | 4 | 3 | 32 | 255 |

- `1, 7` = LDA #7 — load 7 into A
- `2, 5` = LDB #5 — load 5 into B
- `4` = ADD — A = A + B
- `3, 32` = STA @32 — store result at memory address 32
- `255` = HLT

**Go to DASHBOARD and step through:**

- C4 = `0` → Accumulator = 0, everything blank
- C4 = `1` → MNEMONIC: `LDA #7`, Accumulator = `0x07` (7)
- C4 = `2` → MNEMONIC: `LDB #5`, REG_B = `0x05` (5)
- C4 = `3` → MNEMONIC: `ADD`, Accumulator = `0x0C` (12, that is 7+5)
- C4 = `4` → MNEMONIC: `STA @32`, TRACKED MEMORY WRITES shows MEM[0x20] = 12
- C4 = `5` → STATUS: `HALTED`

Result: 7 + 5 = 12, stored in memory address 32.

---

### Program 3 — Countdown loop

**Go to RAM sheet. Change row 0x00 to:**

| +0 | +1 | +2 | +3 | +4 | +5 |
|----|----|----|----|----|-----|
| 1 | 5 | 10 | 13 | 2 | 255 |

- `1, 5` = LDA #5 — load 5 into A
- `10` = DEC — A = A - 1
- `13, 2` = JNZ @2 — if A is not zero, jump back to address 2 (the DEC instruction)
- `255` = HLT

**Go to DASHBOARD and step through:**

- C4 = `0` → Accumulator = 0
- C4 = `1` → MNEMONIC: `LDA #5`, Accumulator = 5
- C4 = `2` → MNEMONIC: `DEC`, Accumulator = 4
- C4 = `3` → MNEMONIC: `JNZ @02`, FLAG_Z = 0, PC jumps back to 0x02
- C4 = `4` → MNEMONIC: `DEC`, Accumulator = 3
- C4 = `5` → MNEMONIC: `JNZ @02`, PC jumps back again
- keep stepping...
- C4 = `11` → MNEMONIC: `DEC`, Accumulator = 0, FLAG_Z flips to 1
- C4 = `12` → MNEMONIC: `JNZ @02`, but Z=1 so jump is skipped, PC moves forward
- C4 = `13` → STATUS: `HALTED`, Accumulator = 0

This is your first loop. DEC ran 5 times by jumping backwards, and the CPU stopped itself when the Zero flag fired.

---

### Program 4 — Subtract two numbers

**Go to RAM sheet. Change row 0x00 to:**

| +0 | +1 | +2 | +3 | +4 | +5 |
|----|----|----|----|----|-----|
| 1 | 9 | 2 | 3 | 5 | 255 |

- `1, 9` = LDA #9
- `2, 3` = LDB #3
- `5` = SUB — A = A - B
- `255` = HLT

**Go to DASHBOARD and step through:**

- C4 = `1` → Accumulator = 9
- C4 = `2` → REG_B = 3
- C4 = `3` → Accumulator = `0x06` (6, that is 9 minus 3)
- C4 = `4` → HALTED

**Bonus:** Try A=3 and B=9 (subtract bigger from smaller). Watch the Accumulator clamp to 0 and FLAG_C flip to 1 — the carry flag telling you the result went negative.

---

### Program 5 — Bitwise AND

**Go to RAM sheet. Change row 0x00 to:**

| +0 | +1 | +2 | +3 | +4 | +5 |
|----|----|----|----|----|-----|
| 1 | 12 | 2 | 10 | 6 | 255 |

- `1, 12` = LDA #12 — binary 00001100
- `2, 10` = LDB #10 — binary 00001010
- `6` = AND — A = A AND B
- `255` = HLT

**Go to DASHBOARD and step through:**

- C4 = `1` → Accumulator = 12 (00001100 in binary)
- C4 = `2` → REG_B = 10 (00001010 in binary)
- C4 = `3` → Accumulator = `0x08` (8 — that is 00001000, only the bits that were 1 in BOTH A and B)
- C4 = `4` → HALTED

Go to the **ALU sheet** and set A=12, B=10. You can see all 8 operations fire at once and the bit-by-bit breakdown at the bottom showing exactly which bits survived the AND.

---

## Peek Inside the Formulas

**Go to CPU ENGINE sheet. Click cell B4.**

You will see the Program Counter formula — the formula that decides where the CPU goes next:

```
=IFS(
  K3=1,   B3,                    // halted, stay put
  I3=255, B3,                    // HLT, stay put
  I3=11,  J3,                    // JMP, go to address
  I3=12,  IF(F3=1, J3, B3+2),   // JZ, jump if zero flag set
  I3=13,  IF(F3=0, J3, B3+2),   // JNZ, jump if zero flag NOT set
  OR(I3=1,I3=2,I3=3), B3+2,     // 2-byte instructions, skip 2
  TRUE(), B3+1                   // everything else, next byte
)
```

**Now click cell C4.**

You will see the ALU formula — the formula that does all arithmetic and logic:

```
=IFS(
  K3=1,   C3,              // halted, preserve A
  I3=1,   J3,              // LDA, load immediate value
  I3=4,   MIN(255,C3+D3),  // ADD, capped at 255
  I3=5,   MAX(0,  C3-D3),  // SUB, floored at 0
  I3=6,   BITAND(C3,D3),   // AND
  I3=7,   BITOR(C3,D3),    // OR
  I3=8,   BITXOR(C3,D3),   // XOR
  I3=9,   MIN(255,C3+1),   // INC
  I3=10,  MAX(0,  C3-1),   // DEC
  TRUE(), C3               // default, preserve A
)
```

B4 moves through instructions. C4 transforms data. Every row from 4 to 66 is the same two formulas, each referencing the row above it. That chain of 64 rows is the processor running.

---

## Sheets Overview

| Sheet | Purpose |
|-------|---------|
| **DASHBOARD** | Main control panel — change STEP in C4, watch the CPU |
| **RAM** | 256-byte memory — edit cells here to load programs |
| **CPU ENGINE** | 64-step execution trace — every row is one clock cycle |
| **ALU** | Interactive arithmetic — change A and B, see all 8 operations live |
| **ISA** | Full instruction reference with opcodes and examples |
| **ASSEMBLER** | Type mnemonics, get byte values to copy into RAM |

---

## License

MIT — use it, teach with it, build on it.
