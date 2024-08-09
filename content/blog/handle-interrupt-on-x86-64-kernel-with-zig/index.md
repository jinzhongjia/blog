---
layout: post
title: Handle Interrupt on x86-64 kernel with zig
date: 2023-11-12T17:37:22Z
tags:
    - zig
    - x86_64
---

## Preface

On x86_64, if we want to handle interrupts, we need to define IDT(Interrupt Descriptor Table), it has 256 elements.

And we also need to set IDTR(Interrupt Descriptor Table Register). In this article, we won’t discuss setting up IDT, that would be too long.

We just discuss how to handle interruptions gracefully, pay attention, what we are talking about here is the interrupt handling function, in other words, what you want to fill in the ide (it only fills part of the idt).

## Tradition

Let we look at osdev(which is the richest os development website), the url is this: [https://wiki.osdev.org/Interrupts_Tutorial](https://wiki.osdev.org/Interrupts_Tutorial).

In osdev, it will tell you to do this, using intel assembly syntax with macro feature:

```nasm
%macro isr_err_stub 1
isr_stub_%+%1:
    call exception_handler
    iret
%endmacro
; if writing for 64-bit, use iretq instead
%macro isr_no_err_stub 1
isr_stub_%+%1:
    call exception_handler
    iret
%endmacro
```

and this code just is a simple handle creator macro, it not store general register status, very unsafe and elegant.

## With zig gracefully

For handling interrupts more precisely, we need to understand what the CPU does when an interrupt occurs, you can see this:{% post_link interrupt-function [interrupt-function] %}

Then we use feature `Naked` function in zig, it will throw away the function calling convention, usually used on inline assembly.

So we can we can get the following function:

```zig
fn generate_handle(comptime num: u8) fn () callconv(.Naked) void {
    const error_code_list = [_]u8{ 8, 10, 11, 12, 13, 14, 17, 21, 29, 30 };

    const public = std.fmt.comptimePrint(
        \\     push ${}
        \\     push %rax
        \\     push %rbx
        \\     push %rcx
        \\     push %rdx
        \\     push %rsp
        \\     push %rbp
        \\     push %rsi
        \\     push %rdi
        \\     push %r8
        \\     push %r9
        \\     push %r10
        \\     push %r11
        \\     push %r12
        \\     push %r13
        \\     push %r14
        \\     push %r15
        \\     mov %rsp, context
    , .{num});

    const save_status = if (for (error_code_list) |value| {
        if (value == num) {
            break true;
        }
    } else false)
        public
    else
        \\     push $0b10000000000000000
        \\
        // Note: the Line breaks are very important
            ++
            public;
    const restore_status =
        \\     mov context, %rsp
        \\     pop %r15
        \\     pop %r14
        \\     pop %r13
        \\     pop %r12
        \\     pop %r11
        \\     pop %r10
        \\     pop %r9
        \\     pop %r8
        \\     pop %rdi
        \\     pop %rsi
        \\     pop %rbp
        \\     pop %rsp
        \\     pop %rdx
        \\     pop %rcx
        \\     pop %rbx
        \\     pop %rax
        \\     add $16, %rsp
        \\     iretq
    ;
    return struct {
        fn handle() callconv(.Naked) void {
            asm volatile (save_status ::: "memory");
            // interruptDispatch just is a higher level interrupt handling function
            asm volatile ("call interruptDispatch");
            asm volatile (restore_status ::: "memory");
        }
    }.handle;
}
```

we use the comptime feature of zig to generate a inline assembly function, the `error_code_list` is an array of EXCEPTION which has error_code.

So we assume that the top layout of the stack is:

```
bottom of stack
--------
SS
------
ESP
------
EFLAGS
------
CS
------
EIP
------
Error Code
------
Interrupt Number
------
General Purpose Registers
--------
top of stack
```

When Interrupt number not has an error code, we will just push a fake error code to stack, here is `$0b10000000000000000`.

Then we will assign `RSP` to `context`(a structure pointer), then call `interruptDispatch` to execute more specific function logic.

After the execution is completed, we try to restore the original register context, so we pop most register, and use `iretq` to return to the original place and continue execution.

Why add 16 to `RSP`, because of maintaining the calling convention of `iretq`.
