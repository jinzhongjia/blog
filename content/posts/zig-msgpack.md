---
date: 2024-02-29T17:47:35+08:00
draft: false
title: Zig Msgpack
tags:
  - zig
  - msgpack
---
# Preface

MessagePack is an efficient binary serialization format. It lets you exchange data among multiple languages like JSON. But it's faster and smaller. Small integers are encoded into a single byte, and typical short strings require only one extra byte in addition to the strings themselves.

It's fast and a typical use case is in neovim and redis! For neovim, it is used to as remote rpc protocol.

*MessagePack spec* : [github](https://github.com/msgpack/msgpack/blob/master/spec.md)

*Zig Msgpack* : [Github](https://github.com/zigcc/zig-msgpack)

## Theory

![](intro.png)

The implementation plan of this protocol is that the header is a one-byte mark to indicate the type of data to be transmitted next. If it is a non-fixed length, it will be followed by a few bytes to indicate the length, and then the data.

Here is a very simple schematic for reading a simple type:

![](msgpack.png)

The types supported:

**Nil**, **Bool**, **Int**, **Float**, **Str**, **Bin**, **Array**, **Map**, **Ext** (Predefined timestamp types)

## Usage

According to [github](https://github.com/zigcc/zig-msgpack) add the package to zig.

**zig-msgpack** provide a generics function call `Pack` to build the read type used.

Like this:

```zig
const bufferType = std.io.FixedBufferStream([]u8);

const pack = msgpack.Pack(
    *bufferType,
    *bufferType,
    bufferType.WriteError,
    bufferType.ReadError,
    bufferType.write,
    bufferType.read,
);
```

Through, we can get a type called `pack`

We use `FixedBufferStream` as the writetype and readtype, `Pack` accepts 6 parameters:

```zig
fn Pack(
    comptime WriteContext: type,
    comptime ReadContext: type,
    comptime WriteError: type,
    comptime ReadError: type,
    comptime writeFn: fn (context: WriteContext, bytes: []const u8) WriteError!usize,
    comptime readFn: fn (context: ReadContext, arr: []u8) ReadError!usize,
) type
```

It looks very similar to `std.io.writer.GenericWriter`and `std.io.GenericReader`, the design of zig-msgpack here references them.

This type `pack` contains methods for reading and writing the msgpack type, we can use the `pack.init(write_context, read_context)` to get a variable.

But since zig does not distinguish strings separately, a separate type `Str` is defined:

```zig
pub const Str = struct {
    str: []const u8,
    pub fn value(self: Str) []const u8 {
        return self.str;
    }
};

/// this is for encode str in struct
pub fn wrapStr(str: []const u8) Str {
    return Str{ .str = str };
}
```


For message pack Bin type:

```zig
pub const Bin = struct {
    bin: []u8,
    pub fn value(self: Bin) []u8 {
        return self.bin;
    }
};

/// this is wrapping for bin
pub fn wrapBin(bin: []u8) Bin {
    return Bin{ .bin = bin };
}
```

For message pack Ext type:

```zig
pub const EXT = struct {
    type: i8,
    data: []u8,
};

/// t is type, data is data
pub fn wrapEXT(t: i8, data: []u8) EXT {
    return EXT{
        .type = t,
        .data = data,
    };
}
```

zig-msgpack provides multiple ways to `write` and `read`, and strict type checking will be performed on the parameters to ensure that no non-read failure errors will occur during runtime.

For example, we write two bool value, and read them:

```zig
var arr: [0xffff_f]u8 = std.mem.zeroes([0xffff_f]u8);
var write_buffer = std.io.fixedBufferStream(&arr);
var read_buffer = std.io.fixedBufferStream(&arr);
var p = pack.init(
    &write_buffer,
    &read_buffer,
);

const test_val_1 = false;
const test_val_2 = true;

try p.write(.{ .bool = test_val_1 });
try p.write(.{ .bool = test_val_2 });

var val_1 = try p.read(allocator);
defer val_1.free(allocator);

var val_2 = try p.read(allocator);
defer val_2.free(allocator);

try std.testing.expect(val_1.bool == test_val_1);
try std.testing.expect(val_2.bool == test_val_2);
```

Overall, we convert the read result into the `Payload` type and obtain the value. This allows us to directly obtain data of unknown structure.

For more examples, check out the unit tests of zig-msgpack.