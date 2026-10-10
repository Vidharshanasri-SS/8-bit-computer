# 8-bit-computer
An FPGA-based 8-bit processor designed in Verilog HDL, featuring a 32×8-bit memory architecture and custom instruction set. It integrates a Control Unit, ALU, registers, program counter, and I/O interface, demonstrating instruction fetch, execution, memory operations, and encryption/decryption on FPGA hardware.

## Overview

This project implements a complete 8-bit computer on an FPGA using Verilog HDL. It includes a datapath (registers, ALU, memory), a multi-cycle control unit, and an input/output interface, all built around a compact custom instruction set of eight instructions.

The processor is intentionally small so that every part of the fetch-decode-execute cycle can be studied, simulated, and observed on real hardware. Its ALU supports XOR and rotate operations, which makes it suitable for demonstrating simple encryption and decryption directly in hardware.

## Key Features

- 8-bit data path and processor architecture
- 32×8-bit RAM for both program and data storage
- Custom instruction set with 8 instructions (3-bit opcode)
- Multi-cycle control unit using timing states T0 to T5
- ALU with XOR and rotate operations
- Input and output interface for interacting with the processor
- Simulated in Vivado and tested on FPGA hardware
- Demonstrates simple encryption and decryption

## Component

| No. | Component               | Abbreviation | Function                                                       |
| --: | ----------------------- | ------------ | -------------------------------------------------------------- |
|   1 | Program Counter         | `PC`         | Holds the address of the next instruction to fetch             |
|   2 | Memory Address Register | `MAR`        | Holds the memory address currently being accessed              |
|   3 | Instruction Register    | `IR`         | Holds the instruction being decoded and executed               |
|   4 | Accumulator             | `ACC`        | Main working register for data and results                     |
|   5 | B Register              | `B`          | Secondary operand register supplying the ALU's second input    |
|   6 | Arithmetic Logic Unit   | `ALU`        | Performs XOR and rotate operations                             |
|   7 | Control Unit            | `CU`         | Generates control signals for each timing state (`T0` to `T5`) |
|   8 | Random Access Memory    | `RAM`        | 32 locations of 8 bits each, holding program and data          |
|   9 | Input/Output Interface  | `I/O`        | Moves data in from and out to the external world               |

## Specifications

| Parameter | Value |
|-----------|-------|
| Data width | 8 bits |
| Memory | 32 × 8-bit RAM |
| Address width | 5 bits (32 locations) |
| Opcode width | 3 bits (8 instructions) |
| Instruction timing | Multi-cycle (`T0` to `T5`) |
| ALU operations | XOR, Rotate |
| Design language | Verilog HDL |
| Toolchain | Xilinx Vivado |
| Target | FPGA development board |

## Instruction Set Architecture

### Instruction Format

Each instruction is 8 bits wide: a 3-bit opcode and a 5-bit address.

```text
  7   6   5    |   4   3   2   1   0
┌──────────────┼───────────────────────┐
│    OPCODE    │        ADDRESS        │
│    3 bits    │         5 bits        │
└──────────────┴───────────────────────┘
```
## Instruction Table

| Opcode | Mnemonic | Description |
|--------|----------|-------------|
| `000` | LDA | Load data from memory into the accumulator |
| `001` | STA | Store the accumulator's contents into memory |
| `010` | XOR | Bitwise XOR operation using the ALU |
| `011` | ROT | Rotate operation using the ALU |
| `100` | JZ | Jump if the zero condition is met |
| `101` | INP | Read data from the input interface |
| `110` | OUT | Send data to the output interface |
| `111` | HLT | Halt the processor |

## Microinstruction Table

| Instruction | T-Cycle | Control Signals | Operation |
|-------------|---------|-----------------|-----------|
| **LDA** | T0 | `CO, MI` | PC address → MAR |
| | T1 | `RO, II` | RAM data → IR, PC increments |
| | T2 | `CE` | PC increments |
| | T3 | `IO, MI` | Address from IR → MAR |
| | T4 | `RO, AI` | RAM data → Accumulator |
| **STA** | T0 | `CO, MI` | PC address → MAR |
| | T1 | `RO, II` | RAM data → IR |
| | T2 | `CE` | PC increments |
| | T3 | `IO, MI` | Address from IR → MAR |
| | T4 | `AO, RI` | Accumulator → RAM |
| **XOR** | T3 | `IO, MI` | Address from IR → MAR |
| | T4 | `RO, BI` | RAM data → B Register |
| | T5 | `XRA, AL0, AI` | A XOR B → Accumulator |
| **JZ** | T3 | `IO, CH` | Jump to address if zero condition is met |
| **OUT** | T3 | `AO, OI` | Accumulator → Output |
| **HLT** | T3 | `HLT` | Halt the processor |
| **INP** | T3 | `IN0, AI` | Input → Accumulator |
| **ROT** | T3 | `ROT, AL0, AI` | Rotate/shift operation → Accumulator |

## Fetch-Decode-Execute Flow

```text
Fetch Instruction from RAM
          ↓
      Load into IR
          ↓
      Decode Opcode
          ↓
        Execute
(LDA / STA / XOR / ROT / JZ / INP / OUT / HLT)
          ↓
       Update PC
          ↓
    Fetch Next Instruction
```

## Example Program: Encryption and Decryption

The ALU uses XOR and rotate operations to implement a simple encryption and decryption process.

## Encryption Flow

1. **INP** → Read plaintext into the Accumulator
2. **XOR** → XOR the data with the key stored in memory
3. **ROT** → Rotate the result to scramble the bits
4. **OUT** → Send the encrypted data to the output

## Decryption Flow

Decryption performs the inverse operations in reverse order:

1. **ROT** → Reverse the rotation
2. **XOR** → XOR with the same key
3. **OUT** → Recover the plaintext

## Example Program T-Cycles Demo

### Encryption Program

The program reads one byte, encrypts it by XORing it with a key and rotating the result, then outputs the ciphertext.

```text
Address: Opcode operand / Data

0  : INP          1010 0000  (A0)
1  : XOR 28       0101 1100  (5C)
2  : ROT          0110 0000  (60)
3  : OUT          1100 0000  (C0)
4  : HLT          1110 0000  (E0)
...
28 : KEY          1010 0101  (A5)
```

**Test values:** Input = `41` (letter "A"), Key = `A5`

### Fetch Cycle

The fetch cycle is common to every instruction.

| T-Cycle | Operation |
|---------|-----------|
| T0 | MAR ← PC |
| T1 | IR ← RAM[MAR] |
| T2 | PC ← PC + 1 |

---

### ◆ Address 0: `INP`

**Fetch Cycle**

| T-Cycle | Operation |
|---------|-----------|
| T0 | MAR ← PC (0) |
| T1 | IR ← RAM[0] (A0) |
| T2 | PC ← 1 |

**Execute Cycle**

| T-Cycle | Operation |
|---------|-----------|
| T3 | ACC ← INPUT (41) |

**Result:**

```text
ACC = 41
PC  = 1
```

---

### ◆ Address 1: `XOR 28`

**Fetch Cycle**

| T-Cycle | Operation |
|---------|-----------|
| T0 | MAR ← PC (1) |
| T1 | IR ← RAM[1] (5C) |
| T2 | PC ← 2 |

**Execute Cycle**

| T-Cycle | Operation |
|---------|-----------|
| T3 | MAR ← IR[4:0] (28) |
| T4 | B ← RAM[28] (A5) |
| T5 | ACC ← ACC XOR B (41 XOR A5 = E4) |

**Result:**

```text
ACC = E4
B   = A5
PC  = 2
```

---

### ◆ Address 2: `ROT`

**Fetch Cycle**

| T-Cycle | Operation |
|---------|-----------|
| T0 | MAR ← PC (2) |
| T1 | IR ← RAM[2] (60) |
| T2 | PC ← 3 |

**Execute Cycle**

| T-Cycle | Operation |
|---------|-----------|
| T3 | ACC ← ROTATE LEFT (ACC) (E4 → C9) |

**Result:**

```text
ACC = C9
PC  = 3
```

---

### ◆ Address 3: `OUT`

**Fetch Cycle**

| T-Cycle | Operation |
|---------|-----------|
| T0 | MAR ← PC (3) |
| T1 | IR ← RAM[3] (C0) |
| T2 | PC ← 4 |

**Execute Cycle**

| T-Cycle | Operation |
|---------|-----------|
| T3 | OUT ← ACC (C9) |

**Result:**

```text
OUTPUT = C9   (ciphertext)
PC     = 4
```

---

### ◆ Address 4: `HLT`

**Fetch Cycle**

| T-Cycle | Operation |
|---------|-----------|
| T0 | MAR ← PC (4) |
| T1 | IR ← RAM[4] (E0) |
| T2 | PC ← 5 |

**Execute Cycle**

| T-Cycle | Operation |
|---------|-----------|
| T3 | HALT (clock stopped) |

**Result:**

```text
Processor halted
```

---

### Encryption Summary

| Step | Instruction | ACC (Binary) | ACC (Hex) |
|------|-------------|--------------|-----------|
| 0 | `INP` | `0100 0001` | `41` |
| 1 | `XOR 28` | `1110 0100` | `E4` |
| 2 | `ROT` | `1100 1001` | `C9` |
| 3 | `OUT` | `1100 1001` | `C9` |

**Plaintext `41` → Ciphertext `C9`**

---

## Decryption Program

Decryption runs the steps in reverse: undo the rotation, then XOR with the same key.

For an 8-bit circular rotate, rotating left by 1 is undone by rotating left 7 more times, because 8 rotations return the value to its original state.

```text
Address: Opcode operand / Data

0  : INP          1010 0000  (A0)
1  : ROT          0110 0000  (60)
2  : ROT          0110 0000  (60)
3  : ROT          0110 0000  (60)
4  : ROT          0110 0000  (60)
5  : ROT          0110 0000  (60)
6  : ROT          0110 0000  (60)
7  : ROT          0110 0000  (60)
8  : XOR 28       0101 1100  (5C)
9  : OUT          1100 0000  (C0)
10 : HLT          1110 0000  (E0)
...
28 : KEY          1010 0101  (A5)
```

**Test values:** Input = `C9` (ciphertext), Key = `A5`

Every instruction uses the same Fetch Cycle (`T0` to `T2`) and the Execute Cycles shown above.

```
```
## Applications

- Learning CPU architecture, including registers, ALU, memory, and control logic
- Understanding the fetch-decode-execute cycle through FPGA implementation
- Demonstrating hardware-level encryption and decryption using XOR and rotate operations
- Providing a foundation for developing larger and more advanced custom processors

## Tools and Hardware

### Development Tools

- **Xilinx Vivado:** 2025.2
- **HDL:** Verilog HDL

### Target Hardware

- **FPGA Board:** Boolean Board
- **FPGA Device:** `XC7S50CSGA324-1`
- **FPGA Family:** Spartan-7

## Results

| Metric                  | Value                  |
| ----------------------- | ---------------------- |
| Simulation              | Verified in Vivado     |
| Hardware Test           | Verified on FPGA board |
| LUT Usage               | 98                     |
| Register Usage          | 51                     |

## Acknowledgements

- Xilinx / AMD for the Vivado design suite
- Digital design and computer architecture course materials and references

## Team Members

* [Vidharshanasri S](https://www.linkedin.com/in/vidharshanasrisivakumar/)
* [Sheeba Angelin N](https://www.linkedin.com/in/sheeba-angelin-n-0a7ab7380/)
* [Harshini M](https://www.linkedin.com/in/harshini-m-997a05380/)



