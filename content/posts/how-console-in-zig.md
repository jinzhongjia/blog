---
layout: post
title: How to console in zig?
date: 2023-02-19 10:39:18
tags:
    - zig
---

# Preface

> Zig is a general-purpose programming language and toolchain for maintaining robust, optimal and reusable software.

## Console

`zig` has very useful standard library, but it not provide a convenient function to console.

Yes, you may say we can use `std.log` or `std.debug.print`, but I dont recommand that.

Let me tell you why?

Zig has no built-in print function or statement.

**Why is there no built-in function like C's `printf`?**

> One answer is that, unlike C, Zig doesn't need to special-case the printing function. In C, `printf` could not be implemented as a normal function without losing its special compile-time checks (i.e., the compiler can check that the number of arguments passed to `printf` matches the number of `%` placeholders in the format string).

About more, you can read this [article](https://zig.news/kristoff/where-is-print-in-zig-57e9)

## Do something

Now, let us wrap a simple buffered print function!

```zig
fn print(comptime str: []const u8) void {
    const stdout_file = std.io.getStdOut().writer();

    var buffer = std.io.bufferedWriter(stdout_file);

    const stdout = buffer.writer();

    stdout.print(str, .{}) catch unreachable;
    buffer.flush() catch unreachable;
}
```

We use stdlib get stdout writter and use bufferedwriter to write string into it, strictly speaking, the code we listed above is actually wrong, because we will flush it at once, and no error handling that means if error occur the whole program will crash!

If we want a no-buffered function, you can just do like this:

```zig
fn print(comptime str: []const u8) void {
    const stdout_file = std.io.getStdOut().writer();
    stdout_file.print(str, .{}) catch unreachable;
}
// this function we also didn't handle error!
```
