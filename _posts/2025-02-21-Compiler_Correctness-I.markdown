---
layout: post
title: "Compiler Correctness - I"
date: 2025-02-21
last_modified_at: 2022-02-21
categories: [Compilers]
tags: [Compiler, Dead Code Removal]
---

_This is a text I made in Summer 2024, while tweaking my own C compiler._

---

As I am coding [my own hand-crafted compiler], I ended up with a small epiphany during this warm afternoon.

From Engineering A Compiler, Cooper & Torczon, Second Edition:

> "After all, if transformations need not preserve meaning, why not replace the entire procedure with a single nop?"

It is generally accepted that using `-O2` or `-O3` optimization levels in compiler might be risky.
I want to show you an example of this statement. Let's take the following function `f`:

```c
int f(int n) {
    for(int i = 10; i == n; i = i) {}
    return 10;
}

int main(){
    return f(10);
}
```

The behaviour of `f` is: _return 10 if the input is not 10, otherwise never end_.

By using -O1 (gcc 11.4.0), this is what you get:

```
_Z1fi:
.L2:
cmpl $10, %edi
je .L2
movl $10, %eax
ret
```

Which is identical to what I have described above. However, by increasing the level of optimization, you end up with:

```
\_Z1fi:
movl $10, %eax
ret
```

Oh no! The compiler thought I don't need the for computation in order to provide the return value.
This is an example of **Dead Code Removal**.
Interestingly enough, the semantics is not preserved, as you can observe by executing the program in the two cases.

I should point out that the compiler is much smarter than we are: I had to pick a weird situation in order to end up in this scenario,
mainly because the behavior depends on a value (`n`) which is not determined at compile time.
When all values are known beforehand, the results are always correct.

How should we interpret this situation? Clearly an endless loop is something that you want to avoid in 99.99% of the cases.
And then, if you are aware of what you are doing, you can ask the compiler to skip some specific optimizations until you get what you need.
However, it's interesting how compiler engineers decided to handle these corner cases by ignoring the basic principle of _keeping the original semantic_.

[my own hand-crafted compiler]: https://github.com/pcineverdies/dummy_cc
[1]: https://www.reddit.com/r/Compilers/comments/176b5oi/how_to_remove_dead_code_in_this_code
[2]: https://outerproduct.net/boring/2023-02-11_term-loop.html
[3]: https://www.r-5.org/files/books/computers/compilers/writing/Keith_Cooper_Linda_Torczon-Engineering_a_Compiler-EN.pdf
