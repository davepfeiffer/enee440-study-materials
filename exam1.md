# ENEE440 Exam #1 Study Guide -- David Pfeiffer

## Topics

- [C to Assembly][]

- [Assembly to C][]

- [Assembly to RTN (Register Transfer Notation)][]

- [AAPCS (ARM Architecture Procedure Call Standard)][]

- [ARM Architecture][]

- [GNU Assembler Macros][]

- [GNU Assembler Directives][]

## C to Assembly and Assembly to C

[C to Assembly]: https://github.com/davepfeiffer/enee440-study-materials/blob/master/exam1.md#c-to-assembly-and-assembly-to-c

[Assembly to C]: https://github.com/davepfeiffer/enee440-study-materials/blob/master/exam1.md#c-to-assembly-and-assembly-to-c

Studying for these topics just requires practice and knowledge of the instruction set (found [here][1]).

## Assembly to RTN

[Assembly to RTN (Register Transfer Notation)]: https://github.com/davepfeiffer/enee440-study-materials/blob/master/exam1.md#assembly-to-rtn

This is very similar to Assembly to C, except register transfer notation might be tricky.

### Overview

- `()` behaves similarly to a dereference in C (`*foo`). It denotes the value stored at the address inside the parenthesis.

- `RHS <- LHS` takes the __value__ on the right hand side and stores it at the __address__ pointed to by the LHS. This behavior can be thought of as an implicit `()` surrounding the LHS.

- Registers are treated as just another address.

- Arithmetic operators/literals are legal and follow the familiar axioms.

### Examples

Translate `ldr r0,[r1,#4]!` from assembly to RTN.

__In RTN:__

```
r0 <- ((r1) + 4)
r1 <- (r1)  + 4
```

__In English:__

- transfer 4 added to the value stored at the __address stored in__ r1 into the address r0.

- transfer 4 added to the value stored at the address r1 (just the registers value) into the address r1.

Both operations happen simultaneously, and r0/r1 are not changed until both operations finish. Think flip-flops and clock cycles.

## ARM Architecture Procedure Call Standard

[AAPCS (ARM Architecture Procedure Call Standard)]: https://github.com/davepfeiffer/enee440-study-materials/blob/master/exam1.md#arm-architecture-procedure-call-standard

In this class we only care about a subset of the [AAPCS][2]. The subset that we care about is as follows:

### Registers

- When arguments are passed to procedures (functions) the are stored in r0-r3 from left to right. There are ways to pass more than 4 arguments but for the purposes of exam 1 we don't care.

- Registers 0-3 are _caller save_. ie: if you want the to keep the values stored in r0-r3 after calling a procedure, they need to be saved elsewhere.

- Registers 4-11 are _callee save_. ie: if you want to use these registers in your procedure, you must save their initial value and restore it before you leave the procedure. This can accomplished by pushing them onto the stack in the function prologue (the instructions needed to setup).

- Registers r12-r15 are special. ie: You shouldn't be using the stack pointer, link register, and program counter for generic operations.

### Prologue

The contents of the prologue (what you need to do to set up), change depending on whether or not you will be calling other functions, using the stack, and/or using r4-r11.

If you are using the stack, you will most likely want to set up a new frame pointer (in r7). Because r7 is _callee save_ it will need to be pushed onto the stack and restored in the epilogue.

If you are calling other functions, you will __always__ want to save the link register so you can return to the function that called you.

_note about stack alignment:_ The stack must be 8 byte aligned at each __external interface__ (call function or return from function). Otherwise it must be word aligned (4 bytes) [3][3].

#### Example

For a non-leaf function (it calls other functions) with 3 arguments and two local variables the prologue would look like the following:

```
foo:
  @ Prologue
  push  {r0, r1, r2, r7, lr}    @ save arguments, frame-pointer, and link reg
  sub   sp,#12                  @ create local vars and pad for 8 byte alignment
  add   r7,sp,#0                @ create new frame-pointer
  @ Body
```

After the prologue, the stack will look like:

```
-----------
| lr      |
-----------
| r7      |
-----------
| r2      |
----------- <- arg2 (FP + 20)
| r1      |
----------- <- arg1 (FP + 16)
| r0      |
----------- <- arg0 (FP + 12)
| padding |
-----------
| x       |
----------- <- x (FP + 4)
| y       |
----------- <- y (frame pointer)
```

### Epilogue

The epilogue only needs to restore callee save registers, clean up the stack, and return to the function that called it.

#### Example

```
foo:
  @ Prologue
  push  {r0, r1, r2, r7, lr}    @ save arguments, frame-pointer, and link reg
  sub   sp,#12                  @ create local vars and pad for 8 byte alignment
  add   r7,sp,#0                @ create new frame-pointer

  @ Body
  @ Do Stuff. Probably calling a function before finishing with r0-3.

  @ Epilogue
  add   sp,#24                  @ throw away local vars and r0, r1, r2
  pop   {r7, pc}                @ restore callee save and return from subroutine
```

After the add instruction the stack will look like:

```
-----------
| lr      |
-----------
| r7      |
----------- <- stack pointer
| r2      |
-----------
| r1      |
-----------
| r0      |
-----------
| padding |
-----------
| x       |
-----------
| y       |
-----------
```

So `pop {r7,pc}` will first put the old frame pointer value back into r7, then put the link register directly into the program counter (same as `pop {r7,lr}` followed by `bx lr`). 

### Note

There are many ways to set up and clean up a function that adhere to the AAPCS. So one size does not fit all. It is important to understand what you can and cannot get away with in function prologue and epilogues.

## ARM Architecture

[ARM Architecture]: https://github.com/davepfeiffer/enee440-study-materials/blob/master/exam1.md#arm-architecture

### Fun (and important) facts about ARM's architecture

- Little Endian: Byte order is least significant first.

- Full Descending Stack: The stack starts at a high address and decreases as you push to it.

- Thumb function calls must have the lsb of their address set to 1 (accomplished with the .thumb_func assembler directive).

- Single address space (Von Neumann).

- Interrupt vectors are at the top of the address space.

- GPIO and other peripherals are memory mapped (aka they are just a place in memory).

- On reset the program counter is initialized to a specific address (the boot vector). The boot vector loads the program text into ram (from flash), initializes the data section, initializes the interrupt table, and jumps into the entry point of the program.

### Op-Code Details

- `mov rn,=literal`

Stores the literal in a literal pool (.ltorg assembler directive) and loads it into rn relative to the program counter. ex: `mov rn,[pc,#relative_offset]`

- `ldr rd,[rs,#off]!`

Pre-increments the source register (rs) and loads from the address stored in it.

- `ldr rd,[rs],#off`

Loads from the address stored in the source register (rs) then increments its contents by off.

- `tbb rt,ro`

Table branch byte requires some memory set up before use. The table register (rt) points to the dispatch table, the offset register (ro) specifies which entry in the table to branch to.

Here's an example for a dispatch table with sparse cases. ie: there are some missing cases in between.

```
dispatch_table:
  .byte #((case_0 - dispatch_table) / 2)
  .byte #((case_1 - dispatch_table) / 2)
  .byte #((default - dispatch_table) / 2)
  .byte #((case_3 - dispatch_table) / 2)
case_0:
  @ do stuff
case_1:
  @ do stuff
case_3:
  @ do stuff
default:
  @ do stuff
```

A few interesting things to note about the example are:

  1. The contents of the entries in the dispatch table are the distance (offset) from the `dispatch_table` address. This makes sense because we know the base address so repeating that information in every entry would be wasteful.

  2. In a similar spirit to the previous detail. The offset is divided by two because ARM requires the LSB of branch address to be set to 1. Explicitly setting the bit to one every time would be wasteful so it is just implicit in the instruction.

  3. The missing case_2 is still filled in with the default case. This action makes sense because leaving the case out would shift the index of the following cases down by 1.

## GNU Assembler Macros

[GNU Assembler Macros]: https://github.com/davepfeiffer/enee440-study-materials/blob/master/exam1.md#gnu-assembler-macros

GNU Assembler macros are slightly more powerful than pure text replacement. The support conditional replacement and recursion. Macros take the following form:

```
.macro MACRO_NAME arg1,arg2,..argN
  @ do stuff
.endm
```

Conditionals take the following form:

```
.if <logical_expression>
  @ the assembler will do this stuff
.else
  @ the assembler will do this stuff
.endif
```

The logical expression is true if it is non-zero.

_note:_ Logical expressions are __not__ register values. Those are only known at runtime and therefore cannot be used at assembly time.

Example:

```
.macro SIMPLE_INC_IF cond, t, f
  .if \cond
    add   \t,#1
  .else
    add   \f,#1
  .endif
.endm
```

- `SIMPLE_INC_IF 0, r0, r1` will translate into `add r1,#1`

- `SIMPLE_INC_IF 123, r0, r1` will translate into `add r0,#1`

- `SIMPLE_INC_IF r0, r1, r2` will not assemble.

## GNU Assembler Directives

[GNU Assembler Directives]: https://github.com/davepfeiffer/enee440-study-materials/blob/master/exam1.md#gnu-assembler-directives

This [cheat-sheet](http://web.mit.edu/gnu/doc/html/as_7.html) will be more complete than whatever I would have written.

[1]: http://www.st.com/content/ccc/resource/technical/document/programming_manual/group0/78/47/33/dd/30/37/4c/66/DM00237416/files/DM00237416.pdf/jcr:content/translations/en.DM00237416.pdf#[{%22num%22%3A1151%2C%22gen%22%3A0}%2C{%22name%22%3A%22XYZ%22}%2C67%2C700%2Cnull]

[2]: infocenter.arm.com/help/topic/com.arm.doc.ihi0042f/IHI0042F_aapcs.pdf

[3]: http://infocenter.arm.com/help/topic/com.arm.doc.faqs/ka4127.html
