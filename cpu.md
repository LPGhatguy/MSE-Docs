# CPU Planning
The base CPU architecture has an implementation for 8-bit, 16-bit, and potentially even 32-bit systems. The game's virtual machine implementation is called *Zarkov*.

## Overview
The base CPU is in the middle of RISC and CISC. Loading into registers is explicit, there is an integrated FPU, and inspiration is drawn from both x86 and ARM.

All CPUs in the game are little-endian.

## Registers
Special Registers:
- Instruction pointer (`rsi`) (N-bit) (0x01)
- Stack pointer (`rss`) (N-bit) (0x02)
- Stack frame pointer (`rsf`) (N-bit) (0x03)
- Comparison register (`rsc`) (8-bit) (0x04)
	- `0x0` - unset
	- `0x1` - equal
	- `0x2` - not equal
	- `0x3` - less
	- `0x4` - greater

General Registers:
- All bits: 4 total: 4x N-bit (`r0-r3`) (0x09-0x0B)
- ` 8-bit`: 8 total: 4x 8-bit (`r4-r7`) (0x0C-0x0F)
- `16-bit`: 12 total: 4x 8-bit (`r4-r7`), 4x 16-bit (`r8-r11`) (0x10-0x13)
- `32-bit`: 16 total: 4x 8-bit (`r4-r7`), 4x 16-bit (`r8-r11`), 4x 32-bit (`r12-r15`) (0x14-0x17)

To access octect `N` of a register, use `r0{N}`. This means that `r8{0}` and `r8{1}` are essentially 8-bit registers of their own.

## Memory
Pointers to data are of the form `[000000]`.

## Constants
Constants are of the form `0x0000`, `255u`, `128`, or `256f`.

## Instructions
- `R#` denotes a register
- `N#` denotes a constant
- `[00]` denotes a pointer
- `(X, X)` denotes possibilties (OR)
- `*` slated for change/removal

All operators store in their first parameter unless otherwise noted.

| Opcode | Description |
| ------ | ----------- |
| **General** |
| `nop` | no operation |
| `cmp R0 R1` | compares `R0` and `R1`, stores in `rsc` |
| **Memory** |
| `mov R0 R1` | move data to `R0` from `R1` |
| `swap R0 R1` | swap the values of `R0` and `R1` |
| `read R0 R1` | load memory into `R0` from pointer `R1` |
| `write R0 R1` | store `R0` into pointer `R1` |
| `put R0 N0` | put `N0` into `R0` |
| **Stack/Functions** |
| `push R0` | push `R0` onto the stack |
| `pop R0` | pop a value off of the stack into `R0` |
| `call R0` | call code at pointer `R0` |
| `ret` | return from function |
| **Jumps** |
| `jmp R0` | jump to pointer `R0` |
| `jet R0` | jump if cmp equal to `R0` |
| `jne R0` | jump if cmp not equal to `R0` |
| `jlt R0` | jump if cmp less than `R0` |
| `jgt R0` | jump if cmp greater than `R0` |
| `jle R0` | jump if cmp less than or equal to `R0` |
| `jge R0` | jump if cmp greater than or equal to `R0` |
| TODO: MORE |
| **Signed Int** |
| `iinc R0` | increment `R0` |
| `idec R0` | decrement `R0` |
| `ineg R0` | negate `R0` |
| `iadd R0 R1` | add `R0` and `R1` |
| `isub R0 R1` | subtract `R0` and `R1` |
| `imul R0 R1` | multiply `R0` and `R1` |
| `idiv R0 R1` | integer divide `R0` and `R1` |
| `ipow R0 R1` | raise `R0` to power `R1` |
| **Unsigned Int** | see int operations |
| `uinc R0` |
| `udec R0` |
| `uadd R0 R1` |
| `usub R0 R1` |
| `umul R0 R1` |
| `udiv R0 R1` |
| `upow R0 R1` |
| **Float (16-bit+ only)** | see int operations |
| `finc R0` |
| `fdec R0` |
| `fneg R0` |
| `fsqrt R0` | square root of `R0` |
| `fadd R0 R1` |
| `fsub R0 R1` |
| `fmul R0 R1` |
| `fdiv R0 R1` |
| **Casting** | casts `R0` to type, puts into `R1` |
| `itou R0 R1` | int to uint |
| `itof R0 R1` | int to float |
| `ftoi R0 R1` | float to int |
| `ftou R0 R1` | float to uint |
| `utof R0 R1` | uint to float |
| `utoi R0 R1` | uint to int |
| **Bitwise** |
| `bnot R0` | bitwise not `R0` |
| `bor R0 R1` | bitwise or `R0` with `R1` |
| `bxor R0 R1` | bitwise xor `R0` with `R1` |
| `band R0 R1` | bitwise and `R0` with `R1` |

