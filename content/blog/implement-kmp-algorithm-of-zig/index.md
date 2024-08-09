---
layout: post
title: Implement KMP algorithm in zig
date: 2023-03-11T09:59:35Z
tags:
    - zig
    - algorithm
---


## Preface

`KMP` called **Knuth–Morris–Pratt string-searching algorithm**, which is the first linear-time algorithm for string matching.

According to wikipedia:

> The algorithm was conceived by James H. Morris and independently discovered by Donald Knuth "a few weeks later" from automata theory. Morris and Vaughan Pratt published a technical report in 1970. The three also published the algorithm jointly in 1977. Independently, in 1969, Matiyasevich discovered a similar algorithm, coded by a two-dimensional Turing machine, while studying a string-pattern-matching recognition problem over a binary alphabet. 

## Principle

The algorithm does not match one by one like other algorithms, it will store some history-matching in a array(we usually call it `next`) to reduce number of traversals.

For example, if we give a string to be matched `ABAABABCAA`, and the character string is `ABABC`, then how can we get the first matched position ?

Of course, you can give me the answer `3`(matched positions start at zero) at once, this is your brain tell you.

Then how can we make computer to do it ?

Let's take a closer look at the strings, when we match part of string, such as `ABA`, then next character is **A** in`ABAABABCAA`. we won't match successfully, then what will do to reduce comparison times.

Notice `ABA`, the longest identical string contained in the prefix string and the suffix string is `A`, that means what ?

Simply put, when we match failed, we don't need to match string from first, we can just mathed from the second character `B`, because the suffix string and prefix string have indetical string `A`, that mean we can just skip one character to continue compare,.

And in comparison, the pointer of `ABAABABCAA` (string to compare) will not go back, so we can just get result in one pass.

Now, we need to build a array called `next` to store the the length of the string that can skip the comparison when there is a match error.

In our brain, the logic is just get the longest indentical string contained in prefix string and suffix string, but in computer we need to chang our brain.

we can just get a pointer which point current position of string`ABABC`, and another digital variable represents the longest length of the prefix and suffix. For first character `A`, we need to store 0 in `next` array, then `AB`, not exist indentical string in prefix and suffix, so store 0, when `ABA`, first charactre `A` and third character `A` is indentical, so we can store 1 in third element.

## Implement

for zig `0.11.0`:

```zig
const std = @import("std");

pub fn main() !void {
    var res = kmp("acbac", "ba");

    std.debug.print("{any}\n", .{res});
}

fn kmp(str: []const u8, patt: []const u8) !usize {
    var gpa = std.heap.GeneralPurposeAllocator(.{}){};
    const allocator = gpa.allocator();

    defer {
        const deinit_status = gpa.deinit();
        //fail test; can't try in defer as defer is executed after we return
        if (deinit_status == .leak) {
            @panic("memory leak");
        }
    }

    var next: []usize = try allocator.alloc(usize, patt.len + 1);
    defer allocator.free(next);

    get_next_val(patt, next);

    var i: usize = 0;

    var prefix_pos: usize = 0;

    while (i < str.len and prefix_pos < patt.len) {
        if (str[i] == patt[prefix_pos]) {
            i = i + 1;
            prefix_pos = prefix_pos + 1;
        } else if (prefix_pos > 0) {
            prefix_pos = next[prefix_pos];
        } else {
            i += 1;
        }
    }

    if (prefix_pos == patt.len) {
        return @intCast(i - prefix_pos);
    }

    return 0;
}

fn get_next_val(patt: []const u8, next: []usize) void {
    @memset(next, 0);

    var i: usize = 1;
    var prefix_pos: usize = 0;

    while (i < patt.len - 1) {
        if (patt[prefix_pos] == patt[i]) {
            prefix_pos += 1;
            next[i] = prefix_pos;
            i += 1;
        } else {
            if (prefix_pos == 0) {
                next[i] = prefix_pos;
                i += 1;
            } else {
                prefix_pos = next[prefix_pos];
            }
        }
    }
}
```
