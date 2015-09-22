# Zarkov CPU Specification
*revision 4*

The Zarkov CPU family comes in 8-bit and 16-bit variants, and is intended to be the primary architecture in the game.

It is little-endian.

## Registers

### Special Registers

- Flag register (`rsf`) (16-bit)
	- bits [0, 15]: reserved
- Instruction pointer (`rsi`) (N-bit)
	- points to the next instruction
- Stack pointer (`rss`) (N-bit)
- Comparison register (`rsc`) (8-bit)
	- `0x0` - unset
	- `0x1` - equal
	- `0x2` - not equal
	- `0x3` - less
	- `0x4` - greater

### General Registers

- `r0-r3`: N-bit
- `r4-r7`: 8-bit
- `r8-r11`: 16-bit, 16-bit systems only

### Register References

Register references are encoded using a single byte. The first 6 bits reference the byte offset of the register (architecture dependent) while the last 2 bits define the length of the register to sample (1, 2, 4, or 8 bytes).

At this time, accessing more than 1 byte from an 8-bit register or 2 bytes from a 16-bit register is invalid.

To access an entire register, use `rN`. The resulting size will be based on the size of the register. To access a smaller sequence of the register, of size `L` bytes, use `rN:L`.

Out-of-bounds access in a register is presently undefined.

To access a section of register `N`, offset by `M` bytes, of byte length `L`, use `rN+M:L`.

This means that the low byte (first) of `r8` can be accessed using `r8:1`, and the high byte can be accessed using `r8+1:1`.

## Memory
Memory is paged in segments that vary based on the bit depth of the processor:

- 8-bit: 64B pages, 4 loaded at once
- 16-bit: 8KiB pages, 8 loaded at once

Memory is loaded into virtual pages from physical pages using the `page` instruction. The maximum number of physical pages for a given depth `b` is `2^b`, yielding a limit of 256 pages for 8-bit systems and 65536 pages for 16-bit systems. This puts an upper limit of 16 KiB of memory for 8-bit systems and 512 MiB of memory for 16-bit systems.

## Memory Watching
It's possible to watch a region of memory for changes and be notified of them. This can be considered a general form of interrupts or exceptions.

This can be done using the `mwatch` instruction. It takes three pointers:

- The output register for the ID of the watcher
- The byte of memory to watch (as a pointer)
- The code to jump to on change (as a pointer)

The watcher ID can later be used for unregistering the watcher using `munwatch`.

## Constants
Constants are of the form `0x0000`, `255u`, `128`, or `256f`. Their size is determined statically based on the size of the register.

`put r4 255` always puts an 8-bit, signed representation of `255` into `r4`.

`put r0 255u` puts an N-bit, unsigned representation of `255` into `r0`.

## Instructions
- `R#` denotes a register
- `N#` denotes a constant (N-length)
- `C#` denotes a constant (register-length)

All operators store results in their first parameter unless otherwise noted.

| Opcode | Description |
| ------ | ----------- |
| **General** |
| `nop` | no operation |
| `cmp R0 R1` | compares `R0` and `R1`, stores in `rsc` |
| **Memory** |
| `mov R0 R1` | copy data to `R0` from `R1` |
| `read R0 R1` | load memory into `R0` from pointer `R1` |
| `write R0 R1` | store `R0` into pointer `R1` |
| `put R0 C0` | put `C0` into `R0` |
| `page N0 R0` | load into page # `N0` from physical page # in `R0` |
| **Stack/Functions** |
| `push R0` | push `R0` onto the stack |
| `pop R0` | pop value off of the stack into `R0` |
| `peek R0 R1` | peek a value from the stack, offset by `R1`, into `R0` |
| `call R0` | call code at pointer `R0` |
| `ret` | return from function |
| **Jumps and Bumps** |
| `jmp R0` | jump to pointer `R0` |
| `bet` | bump on cmp equal |
| `bne` | bump on cmp not equal |
| `blt` | bump on cmp less |
| `bgt` | bump on cmp greater |
| `ble` | bump on cmp less than or equal |
| `bge` | bump on cmp greater than or equal |
| `bsup R0` | bump if feature in `R0` is supported |
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
| `ffloor R0` | rounds `R0` towards negative infinity |
| `fceil R0` | rounds `R0` towards positive infinity |
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
| **Notifications** |
| `mwatch R0 R1 R2` | memory watch byte at pointer `R1`, call to `R2`, put ID in `R0`. <br /> See [Memory Watching](#memory-watching)|
| `munwatch R0` | stop memory watch of ID `R0`. |
| `mpause` | pauses memory watching |
| `mresume` | resumes memory watching |

## Code Samples
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

Jump to address in `r0` on equal
```
put r4 1
put r5 1

cmp r4 r5
bet
jmp r0
```