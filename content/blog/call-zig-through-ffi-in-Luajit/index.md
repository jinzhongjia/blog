---
layout: post
title: Call zig through FFI in Luajit
date: 2023-12-20T18:45:59Z
tags:
    - zig
---

Recently, I have masterd the use of zig, and I accidentally saw an article about luajit's `FFI`. `FFI` can call C function in lua, that inspired me, why not use zig? zig can compile dynamic library which not depend on libc, there will be no compatibility issues.

So, I think this is available, let me try it!

<!--more-->

## How to use FFI in luajit?

In luajit, FFI is an embedded module, we just can use this to import it:

```lua
local ffi = require("ffi")
ffi.cdef[[
  int printf(const char *fmt, ...);
]]

ffi.C.printf("Hello %s!\n", "world")
```

A brief snippet, aha ?

This code is for importing `ffi` and define a function prototype with `ffi.cdef`, then we use `ffi.c.printf` to call clib's function called `printf` to print "Hello, world".

`ffi.c` is a namespace for c standard library prototype, through it we can easily call C standard functions!

### Advanced use

Now, we just can use C standard library, so how to use other library?

em... we need to use `ffi.load` :

```lua
clib = ffi.load(name [,global])
```

the parameter `name` is the dynamic library name, and when `global` is true, the symbol table will be loaded in global namespace. if `name` is a path, it will load dynamic library from the specified path.

OK, this is a brief usage for `FFI`.

## Compile dynamic library in zig

the code in `build.zig`:

```zig
const std = @import("std");

pub fn build(b: *std.Build) void {
    const target = b.standardTargetOptions(.{});

    const optimize = b.standardOptimizeOption(.{});

    const lib = b.addSharedLibrary(.{
        .name = "zig",
        .root_source_file = .{ .path = "src/root.zig" },
        .target = target,
        .optimize = optimize,
    });

    b.installArtifact(lib);
}
```

Then, we just define a export function `hello_world` in zig:

```zig
const std = @import("std");

export fn hello_world() void {
    std.debug.print("Hello, World!\n", .{});
}
```

Compile it, we can get a file called `libzig.so`, copy it to lua's directory.

## FFI call zig

Now, we have a dynamic library compiled by zig!

Let us write lua code:

```lua
local ffi = require("ffi")
local myffi = ffi.load("./libzig.so")

ffi.cdef([[
void hello_world(); /* don't forget to declare */
]])

--- @type function
local hello = myffi.hello_world

hello()
```

Then use luajit to execute this code:

```sh
$ luajit main.lua
Hello, World!
```

Perfect, now we can easily use zig in luajit!
