---
layout: post
title: The differences between CISC and RISC
date: 2023-09-20T21:23:05Z
tags:
    - computer
---

Instruction system is developing in two completely different directions.

One is to strengthen the original command function like adding more complex new instructions to harden software functionality.

The other is to simplify and reduce instructions to speed up the execution of instructions. That is interesting!

# Waht is CISC

_CISC_ is called complex instruction set computer processor, which is developed by **Intel**.

With the cost reduction of hardware and the rising cost of software, that prompts people add more and more complex instructions to instructions system to adapt to various fields. So that constitutes **Complex Instruction Set Computer Processor**!

Main features of CISC:

-   The command system is huge, with generally more than 200 commands!
-   The instruction length is not fixed, there are many formats, and there are many addressing method.
-   There are no restrictions on the instructions that can be accessed from memory.
-   The frequency of using various instructions varies greatly.
-   The time of executing instruction varies greatly, most instructions need several time clock cycle to complete.
-   Most controllers adopt microprogram control. Some instructions are very complex, so that hardwired control cannot be used.
-   It is difficult to generate efficient object code programs through compilation optimization.

Such a huge instruction system places extremely high demands on the design of instructions.

Later, it was discovered that most instructions are rarely used, and only 20% of instructions are used frequently.

So, RISC was born!

# What is RISC

_RISC_ is called Reduced Instruction Set Computer Processor, which is a microprocessor architecture with a simple collection and highly customized set of instructions

The main idea of RISC demands to simplify instruction system, try to use register-register operations instructions, strive to have a consistent instruction format.

Main features of RISC:

-   Select the most frequently used simple instructions, and complex instructions are implemented by a combination of simple instructions.
-   The instruction length is fixed, there are few types of instruction formats, and there are few types of addressing modes.
-   Only load/store instructions access memory, and other instruction operations are performed between registers.
-   Mainly based on hard wiring control, no or less use of micro-program control.
-   Pay attention to compilation optimization and reduce program execution time.

# Advantage

RISC can make full use of the `VLSI` chip area, improve running speed, ease of design and cost reduction, conducive to compiler code optimization.
