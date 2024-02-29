---
layout: post
title: Implement Disjoint-set Data Structure In Zig
date: 2023-03-18T15:43:03Z
tags:
    - zig
    - algorithm
---

# Preface

> In computer science, a disjoint-set data structure, also called a union–find data structure or merge–find set, is a data structure that stores a collection of disjoint (non-overlapping) sets. Equivalently, it stores a partition of a set into disjoint subsets. It provides operations for adding new sets, merging sets (replacing them by their union), and finding a representative member of a set. The last operation makes it possible to find out efficiently if any two elements are in the same or different sets.

# Implement

First, we need to define a sruct for node, the node has three attributes, rank, data, parent.

So our struct now is this:

```zig
const and_lookup = struct {
    nodes: []node_type,

    const Self = @This();
    // you may confuse here, because we will use it in the post.
    // you can just see it like this in c++ or javascript.
}
```

Maybe, we can add something to make it became a `oop` struct. a construct function called `init`, a `deinit` function, a `find` function.

Now, we can add `init` function:

```zig
// init for lookup
pub fn init(arr: []u8) !Self {
    // allocate a array for data
    const data_arr = try allocator.alloc(node_type, arr.len);

    // init the data_arr
    for (data_arr, 0..) |_, i| {
        data_arr[i].parent = @intCast(u8, i);
        data_arr[i].rank = 1;
    }

    // return the struct
    return and_lookup{
        .nodes = data_arr,
    };
}
```

Add `deinit` function:

```zig
// init for lookup
pub fn init(arr: []u8) !Self {
    // allocate a array for data
    const data_arr = try allocator.alloc(node_type, arr.len);

    // init the data_arr
    for (data_arr, 0..) |_, i| {
         data_arr[i].parent = @intCast(u8, i);
        data_arr[i].rank = 1;
    }

    // return the struct
    return and_lookup{
        .nodes = data_arr,
    };
}
```

Add `find` function:

```zig
// find the parent of an element
pub fn find(self: *Self, index: u8) u8 {
    if (self.nodes[index].parent == index) {
        return index;
    } else {
        self.nodes[index].parent = @intCast(u8, self.find(self.nodes[index].parent));
        return self.nodes[index].parent;
    }
}
```

Add `merge` function:

```zig
// merge
pub fn merge(self: *Self, i: u8, j: u8) void {
    var x = @intCast(usize, self.find(i));
    var y = @intCast(usize, self.find(j));
    var nodes = self.nodes;
    if (nodes[x].rank <= nodes[y].rank) {
        nodes[x].parent = @intCast(u8, y);
    } else {
        nodes[y].parent = @intCast(u8, x);
    }

    if (nodes[x].rank == nodes[y].rank and x != y) {
        nodes[y].rank = nodes[y].rank + 1;
    }
}
```

More infomation to be added!
