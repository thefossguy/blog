+++
draft = false
date = 2022-10-06T10:40:00+05:30
title = "Turn an existing mutable variable to immutable in Rust"
description = "This is a series of posts that tries to solve various queries you might have while reading The Book."
slug = "convert_mutable_var_to_immutable_rs.md"
tags = ["the book", "rust", "rust immutable", "rust mutable"]
externalLink = ""
+++

## Introduction

There will be times when you have a mutable variable and want to make it
immutable again. "How do I do that?"

## TELL ME HOW!

Shadowing. Please don't scream again :(

Take the following code snippet for example:

File: test.rs

```rust
fn main() {
    let mut x = 1;
    x = x + 1;
    println!("{x}");
    let x = x;
    x = x + 3;
}
```

And, compiling...

```bash
$ rustc test.rs
error[E0384]: cannot assign twice to immutable variable `x`
 --> test.rs:6:5
  |
5 |     let x = x;
  |         -
  |         |
  |         first assignment to `x`
  |         help: consider making this binding mutable: `mut x`
6 |     x = x + 3;
  |     ^^^^^^^^^ cannot assign twice to immutable variable

error: aborting due to previous error; 1 warning emitted

For more information about this error, try `rustc --explain E0384`.
```

(The unused variable warning from `rustc` is snipped.)

`rustc` is complaining that on line 6, column 5 of "test.rs", it is referring
to `x` as immutable.

But didn't we declare `x` on as `mut` on line 2? Yes, we did.

Now, check the 5<sup>th</sup> line. There I assigned `x` to itself, making use
of Rust's _shadowing_ feature. The name `x` is now pointing to a different
storage that is immutable, but has the value of `&mut x` assigned to it.

{{< notice info >}}
Please note that in the above explanation, I only use `&mut x` to demonstrate
that the _value_ of mutable variable `x` gets copied. This does not mean that
the new `x` will point to `&mut x`.
{{< /notice >}}

## Conclusion

To turn an existing mutable variable into an immutable variable (that Rust
provides a thread-safe guarantee for), you can shadow the existing mutable
variable with a new immutable variable.

```rust
let mut foo; // foo is mutable
let foo = foo; // foo shadows [mutable foo] and is now immutable
```

---

<sub>Please excuse the absence of comments. I do not want to enable any tracking
(except for necessary embeds like YouTube and Twitter). I am not a web-dev and
have no way of manually verifying if a provider tracks you or not.</sub>
