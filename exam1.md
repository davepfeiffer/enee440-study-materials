# ENEE440 Exam #1 Study Guide -- David Pfeiffer

## Topics

- C to Assembly

- Assembly to C

- Assembly to RTN (Register Transfer Notation)

- AAPCS (ARM Architecture Procedure Call Standard)

- ARM Architecture

- GNU Assembler Macros

- GNU Assembler Directives

## C to Assembly and Assembly to C

Studying for these topics just requires practice and knowledge of the instruction set (found [here][1]).

## Assembly to RTN

This is very similar to Assembly to C, except register transfer notation might be tricky.

### Overview

- `()` behaves similarly to a dereference in C (`*foo`). It denotes the value stored at the address inside the parenthesis.

- `RHS <- LHS` takes the __value__ on the right hand side and stores it at the __address__ pointed to by the LHS. This behavior can be thought of as an implicit `()` surrounding the LHS.

- Registers are treated as just another address.

- Arithmetic operators/literals are legal and follow the familiar axioms.

### Examples

Translate `ldr r0,[r1,#4]!` from assembly to RTN.

In RTN:

```

  r0 <- ((r1) + 4)
  r1 <- (r1)  + 4

```

In English:

- transfer the value stored at the __address stored in__ r1 + 4 into the address r0.

- transfer 4 added to the value stored at the address r1 (just the registers value) into the address r1.

Both operations happen simultaneously, and r0/r1 are not changed until both operations finish. Think flip-flops and clock cycles.

[1]: http://www.st.com/content/ccc/resource/technical/document/programming_manual/group0/78/47/33/dd/30/37/4c/66/DM00237416/files/DM00237416.pdf/jcr:content/translations/en.DM00237416.pdf#[{%22num%22%3A1151%2C%22gen%22%3A0}%2C{%22name%22%3A%22XYZ%22}%2C67%2C700%2Cnull]
