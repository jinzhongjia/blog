---
layout: post
title: Assembly in Zig
date: 2023-04-01T21:51:41Z
tags:
    - zig
    - assembly
---

Recently, I want to write a kenel with `zig`, naturally we need to use assembly.

When computer boot, we need to deploy with assembly so we can enter protected mode.

<!--more-->

## Zig reference assembly

### Seprate File

we just need to declare the `assembly function` with keyword `extern`

For example, we next use zig to call assembly to print our most common "Hello, World!"

First, zig itself has its own support for assembly, but now only for `AT&T`, the modish `intel syntax` support poorly.

Now zig use `llvm` for assembly parsing, and zig may have its own assembler in the future.

Now, we config the `build.zig` for assembly:

You can just watch the **/////////////////** symbol

```zig
const std = @import("std");

// Although this function looks imperative, note that its job is to
// declaratively construct a build graph that will be executed by an external
// runner.
pub fn build(b: *std.Build) void {
    // Standard target options allows the person running `zig build` to choose
    // what target to build for. Here we do not override the defaults, which
    // means any target is allowed, and the default is native. Other options
    // for restricting supported target set are available.
    const target = b.standardTargetOptions(.{});

    // Standard optimization options allow the person running `zig build` to select
    // between Debug, ReleaseSafe, ReleaseFast, and ReleaseSmall. Here we do not
    // set a preferred release mode, allowing the user to decide how to optimize.
    const optimize = b.standardOptimizeOption(.{});

    const exe = b.addExecutable(.{
        .name = "zig-prac",
        // In this case the main source file is merely a path, however, in more
        // complicated build scripts, this could be a generated file.
        .root_source_file = .{ .path = "src/main.zig" },
        .target = target,
        .optimize = optimize,
    });

    /////////////////
    // we need to notice here, as long as this sentence can add assembly language support
    exe.addAssemblyFile("./src/hello.s");
    ////////////////

    // This declares intent for the executable to be installed into the
    // standard location when the user invokes the "install" step (the default
    // step when running `zig build`).
    exe.install();

    // This *creates* a RunStep in the build graph, to be executed when another
    // step is evaluated that depends on it. The next line below will establish
    // such a dependency.
    const run_cmd = exe.run();

    // By making the run step depend on the install step, it will be run from the
    // installation directory rather than directly from within the cache directory.
    // This is not necessary, however, if the application depends on other installed
    // files, this ensures they will be present and in the expected location.
    run_cmd.step.dependOn(b.getInstallStep());

    // This allows the user to pass arguments to the application in the build
    // command itself, like this: `zig build run -- arg1 arg2 etc`
    if (b.args) |args| {
        run_cmd.addArgs(args);
    }

    // This creates a build step. It will be visible in the `zig build --help` menu,
    // and can be selected like this: `zig build run`
    // This will evaluate the `run` step rather than the default, which is "install".
    const run_step = b.step("run", "Run the app");
    run_step.dependOn(&run_cmd.step);

    // Creates a step for unit testing.
    const exe_tests = b.addTest(.{
        .root_source_file = .{ .path = "src/main.zig" },
        .target = target,
        .optimize = optimize,
    });

    // Similar to creating the run step earlier, this exposes a `test` step to
    // the `zig build --help` menu, providing a way for the user to request
    // running the unit tests.
    const test_step = b.step("test", "Run unit tests");
    test_step.dependOn(&exe_tests.step);
}
```

Then, we write the `main.zig`:

```zig
const std = @import("std");

extern fn hello_world(?[*:0]const u8) void;

const msg: [:0]const u8 = "Hello World!\n";

pub fn main() void {
    hello_world(msg.ptr);
}
```

the `hello.s`:

Notie: the filename must be `*.s`!

```nasm
.globl hello_world
# global function, expose the hello_world
.type hello_world, @function
# tell compiler, we define a function
.section .text
hello_world:
  mov $4, %eax
  mov $1, %ebx
  mov %edi, %ecx
  # get parameter from register edi, you can learn more on x86-64 abi document
  mov $0xd, %edx
  # the length of string
  int $0x80
  # system call
  ret
```

We alse can get another version for `main.zig`:

notice the function `@ptrToInt`, it can cast ptr to int(usize)

```zig
const std = @import("std");

extern fn hello_world(usize) void;

const msg = "Hello World!\n";

pub fn main() void {
    hello_world(@ptrToInt(msg));
}
```

### Global Assembly

When an assembly expression occurs in a container level comptime block, this is global assembly.

This kind of assembly has different rules than inline assembly. First, volatile is not valid because all global assembly is unconditionally included. Second, there are no inputs, outputs, or clobbers. All global assembly is concatenated verbatim into one long string and assembled together. There are no template substitution rules regarding % as there are in inline assembly expressions.

```zig
const std = @import("std");

comptime {
    asm (
        \\.globl hello_world
        \\.type hello_world, @function
        \\.section .text
        \\hello_world:
        \\  mov $4, %eax
        \\  mov $1, %ebx
        \\  mov %edi, %ecx
        \\  mov $0xd, %edx
        \\  int $0x80
        \\  ret
    );
}

// extern fn hello_world(?[*:0]const u8) void;
extern fn hello_world(usize) void;

// const msg: [:0]const u8 = "Hello World!\n";
const msg = "Hello World!\n";

pub fn main() void {
    // hello_world(msg.ptr);
    hello_world(@ptrToInt(msg));
}
```

### Summary

Then we can just run `zig build run`, you will see "Hello, World!" on your screen!

## Assembly reference zig

wait for completion

## Inline assembly in zig

```zig
pub fn syscall1(number: usize, arg1: usize) usize {
    return asm volatile ("syscall"
        : [ret_reference] "={rax}" (-> usize),
        : [number_reference] "{rax}" (number),
          [arg1_reference] "{rdi}" (arg1),
        : "rcx", "r11"
    );
}
```

In this code, `syscall1` is a wrap function for assembly

Inline assembly is an expression which returns a value, the `asm` keyword begins the expression.

`volatile` is an optional modifier that tells Zig this inline assembly expression has side-effects. Without `volatile`, Zig is allowed to delete the inline assembly code if the result is unused.

`syscall` is assembly instructions.

After the first colon is the output section, `ret_reference` is reference of output, `"={rax}"` is the output constraint string, In this example, the constraint string means "the result value of this inline assembly instruction is whatever is in $rax". `(-> usize)`, it is either a value binding, or `->` and then a type. The type is the result type of the inline assembly expression. If it is a value binding, then `%[ret]` syntax would be used to refer to the register bound to the value.

After the second colon is the output section, `ret_reference` is reference of input, we can have these in the asm string and it would refer to the operands, the register `rax` and register `rdi` will have the value of `number` and `arg1`

After the second colon is the output section, it is the list of clobbers. These declare a set of registers whose values will not be preserved by the execution of this assembly code. These do not include output or input registers. The special clobber value of "memory" means that the assembly writes to arbitrary undeclared memory locations - not only the memory pointed to by a declared indirect output. In this example we list `$rcx` and `$r11` because it is known the kernel syscall does not preserve these registers.

Notice: for now inline asm is limited in zig, constraints don't work, i mean they do but only a limited set. So we avoid them unless you really need to.

## Reference

[Zig Assembly](https://ziglang.org/documentation/master/#Assembly)
