# CPU Planning
The base CPU architecture has an implementation for 8-bit and 16-bit systems. 32-bit systems are in consideration.

The game's virtual machine implementation is called *Zarkov*.

## Overview
The base CPU is intended to be RISC, peppered with a few CISC instructions to minimize compiled size.

All CPUs in the game are little-endian.

## Registers
Special Registers:
- Flag register (`rsf`) (8-bit) (0x0)
	- bit 0: reserved
	- bit 1: reserved
	- bit 2: reserved
	- bit 3: reserved
	- bit 4: reserved
	- bit 5: reserved
	- bit 6: reserved
	- bit 7: reserved
- Instruction pointer (`rsi`) (N-bit) (0x1)
- Stack pointer (`rss`) (N-bit) (0x2)
- Comparison register (`rsc`) (8-bit) (0x3)
	- `0x0` - unset
	- `0x1` - equal
	- `0x2` - not equal
	- `0x3` - less
	- `0x4` - greater

General Registers:
- All bits: 4 total: 4x N-bit (`r0-r3`) (0x4-0x7)
- 8-bit: 8 total: 4x 8-bit (`r4-r7`) (0x8-0xB)
- 16-bit: 12 total: 4x 8-bit (`r4-r7`), 4x 16-bit (`r8-r11`) (0xC-0xF)

In 16-bit implementations, `r0{N}` will access octect `N` of a register. This means that `r8{0}` and `r8{1}` are essentially 8-bit registers of their own.

Register references are encoded using a single byte. The first four bits select the register, and the second four bits select the portion of the register to select.

A second four bits of `0x0` selects the entire register, `0x1` selects the first octect, and `0x2` selects the second octet.

`r0` would be encoded as `0x40`

`r3{1}` would be encoded as `0x72`

## Memory
Memory is paged in segments that vary based on the bit depth of the processor:
- 8-bit: 64B pages, 4 loaded at once
- 16-bit: 8KiB pages, 8 loaded at once

Memory is loaded into virtual pages from physical pages using the `page` instruction. The maximum number of physical pages for a given depth `b` is `2^b`, yielding a limit of 256 pages for 8-bit systems and 65536 pages for 16-bit systems.

## Constants
Constants are of the form `0x0000`, `255u`, `128`, or `256f`. Their size is determined statically and contextually.

`put r4 255u` always puts an 8-bit, unsigned representation of `255` into `r4`.

`put r0 255u` puts an N-bit, unsigned representation of `255` into `r0`, little-endian.

## Instructions
- `R#` denotes a register
- `N#` denotes a constant
- `(X, X)` denotes possibilities (OR)

All operators store in their first parameter unless otherwise noted.

| Opcode | Description |
| ------ | ----------- |
| **General** |
| `nop` | no operation |
| `cmp R0 R1` | compares `R0` and `R1`, stores in `rsc` |
| **Memory** |
| `mov R0 R1` | move data to `R0` from `R1` |
| `read R0 R1` | load memory into `R0` from pointer `R1` |
| `write R0 R1` | store `R0` into pointer `R1` |
| `put R0 N0` | put `N0` into `R0` |
| `page N0 R0` | load into page # `N0` from physical page # in `R0` |
| **Stack/Functions** |
| `push R0` | push `R0` onto the stack |
| `pop R0` | pop value off of the stack into `R0` |
| `peek R0 N0` | peek a value from the stack, offset by `N0`, into `R0` |
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
| **Float (16-bit only)** | see int operations |
| `finc R0` |
| `fdec R0` |
| `fneg R0` |
| `fsqrt R0` | square root of `R0` |
| `fadd R0 R1` |
| `fsub R0 R1` |
| `fmul R0 R1` |
| `fdiv R0 R1` |
| `fpow R0 R1` |
| `ffloor R0` |
| `fceil R0` |
| **Casting** | casts `R1` to type, puts into `R0` |
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

## Samples
Swap two registers
```
put r4 255u
put r5 127

push r4
mov r5 r4
pop r5
```

Call function (`__cdecl`)
```
put r0 0xF00D
put r4 255u

push r4
call r0
pop r4
```

Check if a system flag is set
```
put r4 0x01
mov r4 r5
band r5 rsf
cmp r4 r5
```

Do some floating-point calculations
```
put r4 255u

utof r8 r4
put r9 2f
put r10 1.5f
fpow r8 r9
fdiv r8 r10
fceil r8
ftou r4 r8
```