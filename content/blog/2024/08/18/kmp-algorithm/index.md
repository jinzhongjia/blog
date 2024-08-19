+++
title = "KMP Algorithm"
author = ["jinzhongjia"]
date = 2024-08-18
layout = "blog"
lastmod = 2024-08-19T09:43:59+08:00
draft = false
math = false
+++

## KMP Algorithm {#kmp-algorithm}

> In computer science, the Knuth-Morris-Pratt (KMP) string search algorithm is used to find the occurrence of a word `W` within a string `S`. The algorithm takes advantage of the fact that when a mismatch occurs, the word itself contains enough information to determine the next possible starting position for a match. This characteristic allows the algorithm to avoid rechecking previously matched characters.
>
> <!--more-->
>
> The algorithm was conceived by Donald Knuth and Vaughan Pratt in 1974, and the same year, James H. Morris independently designed it. The three researchers published their work together in 1977.


### Principle {#principle}

Unlike other algorithms that match characters one by one, the KMP algorithm stores some historical matches in an array (commonly referred to as the next array) to reduce the number of comparisons.

For example, if we have a string to match, `ABAABABCAA` , and a character string `ABABC` , how do we determine the first match position?

Of course, you might immediately think of the answer as `3` (considering zero-based indexing), which is what your brain tells you.

So how do we make the computer do this?

Let’s take a closer look at these strings. When we match the substring ABA, the next character in `ABAABABCAA` is `A`. Since this does not result in a match, how can we reduce the number of comparisons?

Notice that the longest common substring contained in both the prefix and suffix of `ABA` is `A`. What does this mean?

In simple terms, when we encounter a mismatch, we do not need to start matching from the beginning of the string again. Instead, we can start matching from the second character `B`, because the suffix and prefix share the common substring `A`. This allows us to skip one character and continue the comparison.

In this comparison, the pointer for `ABAABABCAA` (the string being compared) does not backtrack, allowing us to achieve results in a single pass.

Thus, it becomes clear that the essence of KMP is not just the convenience of comparison, but rather the storage of information in the next array.

Now, let’s construct an array called `next` to store the lengths of substrings that can be skipped during mismatches.

In our minds, the logic is simply to obtain the longest common substring contained in both the prefix and suffix, but in the computer, we need to alter our way of thinking.

We can maintain a pointer pointing to the current position in the string `ABABC`, along with another variable representing the length of the longest prefix and suffix. For the first character `A`, we store `0` in the next array. For `AB`, since there are no common substrings in the prefix and suffix, we store `0`. When we reach `ABA`, the first character `A` and the third character `A` are the same, so we can store `1` in the third element.

This process essentially describes how the next array is generated, but we need to express it in a programming language!


### The next Array {#the-next-array}

`next[i]` represents the length of the longest equal prefix and suffix in the prefix `P[0...i-1]` of the pattern string `P`.

Definitions of prefix and suffix are as follows:

Prefix: The beginning part of the string, excluding the entire string.

Suffix: The ending part of the string, excluding the entire string.


### Constructing the Array {#constructing-the-array}


#### Initialization: {#initialization}

Create a next array of length equal to the pattern string, initialized to `0` .
Set two pointers: `i` for traversing the pattern string (starting from `1`), and `j` for recording the length of the prefix (starting from `0`).


#### Traversing the Pattern String: {#traversing-the-pattern-string}

When `P[i]` equals `P[j]`, it indicates a longer prefix and suffix has been found:
Update next `[i] = j + 1`, representing the length of the longest prefix and suffix for the current character.

Increment both `j` and `i` by `1` to continue comparing the next character.

When `P[i]` does not equal `P[j]`:

If `j` is not `0`, it indicates that we can backtrack using the next array, updating `j` to `next[j-1]` and continuing the comparison.

If `j` is `0`, it indicates there are no equal prefixes and suffixes, so set `next[i] = 0` and increment `i` by `1`.


### Code Implementation {#code-implementation}

Here is the implementation of the KMP algorithm in Python:

```python
def compute_next(pattern):
    m = len(pattern)
    next = [0] * m  # Initialize the next array
    j = 0  # j is the length of the prefix
    for i in range(1, m):
        while j > 0 and pattern[i] != pattern[j]:  # When not matching
            j = next[j - 1]  # Backtrack
        if pattern[i] == pattern[j]:  # When matching
            j += 1  # Increase the prefix length
        next[i] = j  # Update next[i]
    return next

def kmp_search(text, pattern):
    n = len(text)
    m = len(pattern)
    next = compute_next(pattern)  # Compute the next array
    j = 0  # Pointer for the pattern string

    for i in range(n):  # Traverse the text string
        while j > 0 and text[i] != pattern[j]:  # When not matching
            j = next[j - 1]  # Backtrack
        if text[i] == pattern[j]:  # When matching
            j += 1  # Increase the pattern string pointer
        if j == m:  # Match found
            print(f"Pattern '{pattern}' found in text at position: {i - m + 1}")
            j = next[j - 1]  # Continue searching for the next match

# Example usage
text = "ABABDABACDABABCABAB"
pattern = "ABABCABAB"
kmp_search(text, pattern)
```
