---
layout: post
title: Implement tree and forest in zig
date: 2023-03-16T20:42:37Z
tags:
    - zig
    - - algorithm
---

Recently, I review your knowledge about trees and forests, and now I try implement them in zig !

The full code can be found [here](https://gist.github.com/jinzhongjia/a624722848f4cf482254ab18fbe5961e) !

<!--more-->

## Implement

Fist, we need to define the node, for convenice, we call it `TreeNode`.

```zig
const TreeNode = struct {
    data: u8,
    l_child: ?*TreeNode,
    // optional type, now l_child could be "*TreeNode" and "null"
    r_child: ?*TreeNode,
    // optional type, now r_child could be "*TreeNode" and "null"
};
```

Now we need to fake some fake data, for example a binary tree.

But we don't need to write all nodes by hand, just write a construct function, uh-huh?

```zig
fn new_node(data: u8, l_Child: ?*TreeNode, r_child: ?*TreeNode) !*TreeNode {
    var node = try allocator.create(TreeNode);
    // try is just a syntactic sugar, it equal cathch |err| { return err; }
    node.data = data;
    node.l_child = l_Child;
    node.r_child = r_child;
    return node;
}
```

Now, we can just make node easliy, here is code!

```zig
var node_2 = try new_node(2, null, null);
var node_3 = try new_node(3, null, null);
var node_root = try new_node(1, node_2, node_3);
```

### Antecedent traversal of binary trees:

For this, we need to use `std.ArrayList`, ths direction is according to `master` document.

> A contiguous, growable list of items in memory. This is a wrapper around an array of T values. Initialize with init.
>
> This struct internally stores a std.mem.Allocator for memory management. To manually specify an allocator with each method call see ArrayListUnmanaged.

A quick look at how to use arraylist:

```zig
// std.ArrayList is a useful type
// the allocator is memory allocator, such as std.heap.GeneralPurposeAllocator
var gpa = std.heap.GeneralPurposeAllocator(.{}){};
const allocator = gpa.allocator();

var list = std.ArrayList(*TreeNode).init(allocator);
defer list.deinit();

// it can be used as stack
try list.append(node_root);
list.popOrNull();// just get the latest element and return it, if not exist, it will reutrn null.
```

For convenience, we can define a function called `do_somethind`:

```zig
fn do_something(data: u8) void {
    std.log.info("element is {}\n", .{data});
}
```

For mare information, you can learn it from [here](https://ziglang.org/documentation/master/std/#A;std:ArrayList)

Main logic of pre order traversal:

```zig
fn pre_order_traversal(list: *std.ArrayList(*TreeNode)) !void {
    // why we need to pass pointer ?
    // because if we pass a variable of arraylist, it will automatically switch into const pointer
    // but the function only need pointer, not const!!!
    var last = list.*.popOrNull();
    if (last != null) {
        do_something(last.?.data);
        if (last.?.l_child != null) {
            try list.*.append(last.?.l_child.?);
            try pre_order_traversal(list);
        }
        if (last.?.r_child != null) {
            try list.*.append(last.?.r_child.?);
            try pre_order_traversal(list);
        }
    }
}
```

we can easily implement the `in order traversal` and `post orde traversal`, here is main logic:

```zig
// in order traversal
fn in_order_traversal(list: *std.ArrayList(*TreeNode)) !void {
    var last = list.*.popOrNull();
    if (last != null) {
        if (last.?.l_child != null) {
            try list.*.append(last.?.l_child.?);
            try pre_order_traversal(list);
        }
        do_something(last.?.data);
        if (last.?.r_child != null) {
            try list.*.append(last.?.r_child.?);
            try pre_order_traversal(list);
        }
    }
}
```

```zig
// post order traversal
fn post_order_traversal(list: *std.ArrayList(*TreeNode)) !void {
    var last = list.*.popOrNull();
    if (last != null) {
        if (last.?.l_child != null) {
            try list.*.append(last.?.l_child.?);
            try pre_order_traversal(list);
        }
        if (last.?.r_child != null) {
            try list.*.append(last.?.r_child.?);
            try pre_order_traversal(list);
        }
        do_something(last.?.data);
    }
}
```
