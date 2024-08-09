---
layout: post
title: Registers in x86 operating system
date: 2023-05-27T00:46:28Z
tags:
    - os
---

# Preface

Recently I try to implement a 32-bit microkernel with zig, but I always forget the purpose of register, so record here.

## General Purpose Registers

| 64-bit | 32-bit | 16-bit | 8 high bits | 8 low bits | Description        |
| ------ | ------ | ------ | ----------- | ---------- | ------------------ |
| RAX    | EAX    | AX     | AH          | AL         | Accumulator        |
| RBX    | EBX    | BX     | BH          | BL         | Base               |
| RCX    | ECX    | CX     | CH          | CL         | Counter            |
| RDX    | EDX    | DX     | DH          | DL         | Data               |
| RSI    | ESI    | SI     | N/A         | SIL        | Source             |
| RDI    | EDI    | DI     | N/A         | DIL        | Destination        |
| RSP    | ESP    | SP     | N/A         | SPL        | Stack Pointer      |
| RBP    | EBP    | BP     | N/A         | BPL        | Stack Base Pointer |

<br />

## Pointer Registers

| 64-bit | 32-bit | 16-bit | Description         |
| ------ | ------ | ------ | ------------------- |
| RIP    | EIP    | IP     | Instruction Pointer |

<br />

## Segment Registers

| 16-bit | Description               |
| ------ | ------------------------- |
| CS     | Code Segment              |
| DS     | Data Segment              |
| ES     | Extra Segment             |
| SS     | Stack Segment             |
| FS     | General Purpose F Segment |
| GS     | General Purpose G Segment |

<br />

## Reference

- [CPU Registers x86](https://wiki.osdev.org/CPU_Registers_x86)