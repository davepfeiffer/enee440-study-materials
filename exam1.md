# ENEE440 Exam #1 Study Guide -- David Pfeiffer

## Topics

- [C to Assembly][]

- [Assembly to C][]

- [Assembly to RTN (Register Transfer Notation)][]

- AAPCS (ARM Architecture Procedure Call Standard)

- ARM Architecture

- GNU Assembler Macros

- GNU Assembler Directives

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

In this class we only care about a subset of the [AAPCS][2]. The subset that we care about is as follows:

### Registers

- When arguments are passed to procedures (functions) the are stored in r0-r3 from left to right. There are ways to pass more than 4 arguments but we don't care.

- Registers 0-3 are _caller save_. ie: if you want the to keep the values stored in r0-r3 after calling a procedure, they need to be saved elsewhere.

- Registers 4-11 are _callee save_. ie: if you want to use these registers in your procedure, you must save their initial value and restore it before you leave the procedure. This can accomplished by pushing them onto the stack in the function prologue (the instructions needed to setup).

- Registers r12-r15 are special. ie: You shouldn't be using the stack pointer, link register, and program counter for generic operations.

### Prologue



[1]: http://www.st.com/content/ccc/resource/technical/document/programming_manual/group0/78/47/33/dd/30/37/4c/66/DM00237416/files/DM00237416.pdf/jcr:content/translations/en.DM00237416.pdf#[{%22num%22%3A1151%2C%22gen%22%3A0}%2C{%22name%22%3A%22XYZ%22}%2C67%2C700%2Cnull]

[2]: infocenter.arm.com/help/topic/com.arm.doc.ihi0042f/IHI0042F_aapcs.pdf
