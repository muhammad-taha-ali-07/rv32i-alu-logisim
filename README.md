# RV32I 32-bit ALU in Logisim Evolution

A 32-bit RV32I Arithmetic Logic Unit (ALU) designed in **Logisim Evolution** with support for R-type, I-type, shift, comparison, logical, LUI, and AUIPC instructions using automatic instruction decoding and ALU control.

## Overview

This project implements a 32-bit RISC-V ALU based on the **RV32I instruction set architecture**.

The circuit accepts a 32-bit RISC-V instruction, extracts the required instruction fields, determines the ALU operation automatically, selects the appropriate operands, and generates a 32-bit result.

The project was developed using **Logisim Evolution 4.1.0**.

## Features

- 32-bit ALU datapath
- R-type instruction support
- I-type instruction support
- Automatic instruction decoding
- Automatic ALU operation selection
- 4-bit ALU control
- Priority Encoder based operation selection
- 32-bit immediate sign extension
- Automatic selection between register B and immediate operand
- Signed comparison
- Unsigned comparison
- Logical left shift
- Logical right shift
- Arithmetic right shift
- Shift-immediate support
- LUI support
- AUIPC support
- 32-bit result output

## Supported Instructions

| Operation | Instructions |
|---|---|
| Addition | `ADD`, `ADDI` |
| Subtraction | `SUB` |
| Shift Left Logical | `SLL`, `SLLI` |
| Set Less Than | `SLT`, `SLTI` |
| Set Less Than Unsigned | `SLTU`, `SLTIU` |
| XOR | `XOR`, `XORI` |
| Shift Right Logical | `SRL`, `SRLI` |
| Shift Right Arithmetic | `SRA`, `SRAI` |
| OR | `OR`, `ORI` |
| AND | `AND`, `ANDI` |
| Load Upper Immediate | `LUI` |
| Add Upper Immediate to PC | `AUIPC` |

## ALU Control Codes

The final ALU multiplexer uses a 4-bit control signal.

| ALU Control | Operation |
|---|---|
| `0000` | ADD |
| `0001` | SUB |
| `0010` | SLL |
| `0011` | SLT |
| `0100` | SLTU |
| `0101` | XOR |
| `0110` | SRL |
| `0111` | SRA |
| `1000` | OR |
| `1001` | AND |
| `1010` | LUI |
| `1011` | AUIPC |

## Instruction Format

The circuit accepts a **32-bit RISC-V instruction**.

Important fields are extracted from the instruction:

```text
Opcode  = Instruction[6:0]

funct3  = Instruction[14:12]

funct7  = Instruction[31:25]

For R-type instructions:

funct7 + funct3
7 bits    3 bits
   \       /
    \     /
     10-bit
   combined value

The circuit concatenates funct7 and funct3 into a 10-bit value.

This value is compared with predefined instruction patterns to identify the required ALU operation.

R-Type Instruction Decoding

The following 10-bit values are used for R-type instructions:

Instruction	funct7	funct3	Combined
ADD	0000000	000	0000000000
SUB	0100000	000	0100000000
SLL	0000000	001	0000000001
SLT	0000000	010	0000000010
SLTU	0000000	011	0000000011
XOR	0000000	100	0000000100
SRL	0000000	101	0000000101
SRA	0100000	101	0100000101
OR	0000000	110	0000000110
AND	0000000	111	0000000111
I-Type Instruction Decoding

I-type arithmetic instructions use:

Opcode = 0010011

or hexadecimal:

0x13

The funct3 field determines the required operation.

Instruction	funct3
ADDI	000
SLLI	001
SLTI	010
SLTIU	011
XORI	100
SRLI / SRAI	101
ORI	110
ANDI	111
Immediate Generation

For normal I-type arithmetic instructions, the immediate is extracted from:

Instruction[31:20]

This provides a 12-bit immediate.

The immediate is then sign-extended:

12-bit Immediate
       |
       v
 Sign Extension
       |
       v
32-bit Immediate

The resulting 32-bit value is used as the second ALU operand.

ALUSrc Multiplexer

A 32-bit multiplexer selects the second ALU operand.

For R-type instructions:

ALUSrc = 0

ALU_B = Register B

For I-type instructions:

ALUSrc = 1

ALU_B = Sign-Extended Immediate

The I-type opcode comparator automatically generates the ALUSrc signal.

Shift Amount Selection

RISC-V RV32I shift operations use a 5-bit shift amount.

For R-type instructions such as:

SLL
SRL
SRA

the shift amount comes from the normal shift input.

For shift-immediate instructions:

SLLI
SRLI
SRAI

the shift amount is extracted from:

Instruction[24:20]

which produces:

shamt[4:0]

A 5-bit multiplexer selects between:

R-type shift amount

and:

Instruction[24:20]
LUI

LUI stands for:

Load Upper Immediate

The instruction places the immediate into the upper 20 bits of the result.

LUI Result = Immediate << 12

Example:

Immediate = 00000000000000000001

Result =
00000000000000000001000000000000
AUIPC

AUIPC stands for:

Add Upper Immediate to PC

The operation is:

AUIPC Result = PC + (Immediate << 12)

The circuit reuses the upper-immediate output and adds it to the 32-bit Program Counter.

Priority Encoder

Each decoded instruction produces a 1-bit detection signal.

Examples:

ADD_Detect
SUB_Detect
SLL_Detect
SLT_Detect
XOR_Detect
AND_Detect
...

These signals are connected to a Priority Encoder.

The Priority Encoder generates the 4-bit ALU control code.

Example:

ADD  -> 0000
SUB  -> 0001
SLL  -> 0010
SLT  -> 0011
...

The ALU control signal is connected to the main result multiplexer.

ALU Architecture

The simplified architecture is:

                  32-bit Instruction
                         |
                         v
                Instruction Splitters
                         |
              +----------+----------+
              |                     |
           Opcode              funct3/funct7
              |                     |
              +----------+----------+
                         |
                         v
                Instruction Decoder
                         |
                         v
                  Priority Encoder
                         |
                   4-bit Control
                         |
                         v
                   +-----------+
 A ---------------->|           |
                    |  32-bit   |
 B / Immediate ---->|    ALU    |
                    |           |
                    +-----+-----+
                          |
                          v
                    Result MUX
                          |
                          v
                    32-bit Result
ALU Operations

The circuit contains separate functional units for:

ADD
SUB
SLL
SLT
SLTU
XOR
SRL
SRA
OR
AND
LUI
AUIPC

All operation outputs are connected to the main multiplexer.

The decoded ALU control signal determines which result is sent to the final output.

Testing

The circuit has been tested with multiple R-type and I-type operations.

Example SLLI Test
A = 00000000000000000000000000000101

Shift Amount = 2

The instruction provides:

shamt = 00010
funct3 = 001
opcode = 0010011

Result:

00000000000000000000000000010100

This corresponds to:

5 << 2 = 20

Therefore, the SLLI operation is working correctly.

Example XORI Test

For:

A = 5
Immediate = 3

Binary values:

A         = 0101
Immediate = 0011

XOR operation:

0101
0011
----
0110

Therefore:

5 XOR 3 = 6

32-bit result:

00000000000000000000000000000110
Software

Built using:

Logisim Evolution 4.1.0

Project File
RV32I_32bit_ALU.circ
Repository

Recommended GitHub repository name:

rv32i-alu-logisim
Repository Description

32-bit RV32I ALU in Logisim Evolution with automatic instruction decoding and support for arithmetic, logic, shift, comparison, LUI, and AUIPC operations.

Status

Most implemented instructions have been tested successfully.

 R-type ALU operations
 Immediate arithmetic operations
 ADDI
 SLTI
 SLTIU
 XORI
 ORI
 ANDI
 LUI
 AUIPC
 SLLI
 Final verification of SRLI
 Final verification of SRAI
Future Improvements

Possible future improvements include:

Full register file implementation
Program Counter integration
Complete RV32I processor datapath
Branch instruction support
Load/store instruction support
Automatic register selection using rs1, rs2, and rd
Control Unit implementation
Memory interface
Full single-cycle RISC-V processor design
Author

Muhammad Taha Ali

Computer Systems Engineering


That version is much more complete and GitHub-ready. Once **SRLI and SRAI pass**, just change the last two status lines from `[ ]` to `[x]`. 🔥
