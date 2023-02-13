# RISC-U

[![Build Statue](https://img.shields.io/github/workflow/status/cksystemsgroup/riscu/Test)](https://github.com/cksystemsgroup/riscu/actions)
[![Crate](https://img.shields.io/crates/v/riscu.svg)](https://crates.io/crates/riscu)
[![API](https://docs.rs/riscu/badge.svg)](https://docs.rs/riscu)
![Rust Version](https://img.shields.io/badge/Rust-v1.60.0-yellow)
[![Lines of Code](https://img.shields.io/tokei/lines/github/cksystemsgroup/riscu)](https://github.com/cksystemsgroup/riscu)
[![License](https://img.shields.io/crates/l/riscu)](https://github.com/cksystemsgroup/riscu/blob/master/LICENSE)

A simple library for loading and decoding of ELF64 RISC-U files. This binaries are generated by [Selfie](https://github.com/cksystemsteaching/selfie), an educational software system of a tiny self-compiling C compiler, a tiny self-executing RISC-V emulator, and a tiny self-hosting RISC-V hypervisor.

## RISC-U Instruction Set
RISC-U is a tiny subset of the 64-bit [RISC-V](https://en.wikipedia.org/wiki/RISC-V) instruction set. The selfie system implements a compiler that targets RISC-U as well as a RISC-U emulator that interprets RISC-U code. RISC-U consists of just 14 instructions listed below. For details on the exact encoding, decoding, and semantics of RISC-U code see the selfie implementation.

### Machine

A RISC-U machine has a 64-bit program counter denoted `pc`, 32 general-purpose 64-bit registers numbered `0` to `31` and denoted `zero`, `ra`, `sp`, `gp`, `tp`, `t0`-`t2`, `s0`-`s1`, `a0`-`a7`, `s2`-`s11`, `t3`-`t6`, and 4GB of byte-addressed memory.

Register `zero` always contains the value 0. Any attempts to update the value in `zero` are ignored.

### Instructions

RISC-U instructions are encoded in 32 bits (4 bytes) each and stored next to each other in memory such that there are two instructions per 64-bit double word. Memory, however, can only be accessed at 64-bit double-word granularity.

The parameters `rd`, `rs1`, and `rs2` used in the specification of the RISC-U instructions below may denote any of the 32 general-purpose registers.

The parameter `imm` denotes a signed integer value represented by a fixed number of bits depending on the instruction.

#### Initialization

`lui rd,imm`: `rd = imm * 2^12; pc = pc + 4` with `-2^19 <= imm < 2^19`

`addi rd,rs1,imm`: `rd = rs1 + imm; pc = pc + 4` with `-2^11 <= imm < 2^11`

#### Memory

`ld rd,imm(rs1)`: `rd = memory[rs1 + imm]; pc = pc + 4` with `-2^11 <= imm < 2^11`

`sd rs2,imm(rs1)`: `memory[rs1 + imm] = rs2; pc = pc + 4` with `-2^11 <= imm < 2^11`

#### Arithmetic

`add rd,rs1,rs2`: `rd = rs1 + rs2; pc = pc + 4`

`sub rd,rs1,rs2`: `rd = rs1 - rs2; pc = pc + 4`

`mul rd,rs1,rs2`: `rd = rs1 * rs2; pc = pc + 4`

`divu rd,rs1,rs2`: `rd = rs1 / rs2; pc = pc + 4` where `rs1` and `rs2` are unsigned integers.

`remu rd,rs1,rs2`: `rd = rs1 % rs2; pc = pc + 4` where `rs1` and `rs2` are unsigned integers.

#### Comparison

`sltu rd,rs1,rs2`: `if (rs1 < rs2) { rd = 1 } else { rd = 0 } pc = pc + 4` where `rs1` and `rs2` are unsigned integers.

#### Control

`beq rs1,rs2,imm`: `if (rs1 == rs2) { pc = pc + imm } else { pc = pc + 4 }` with `-2^12 <= imm < 2^12` and `imm % 2 == 0`

`jal rd,imm`: `rd = pc + 4; pc = pc + imm` with `-2^20 <= imm < 2^20` and `imm % 2 == 0`

`jalr rd,imm(rs1)`: `tmp = ((rs1 + imm) / 2) * 2; rd = pc + 4; pc = tmp` with `-2^11 <= imm < 2^11`

#### System

`ecall`: system call number is in `a7`, parameters are in `a0-a2`, return value is in `a0`.

## License

Copyright (c) 2020, [the Selfie authors](https://github.com/cksystemsteaching/selfie). All rights reserved.

Licensed under the [MIT](LICENSE) license.
