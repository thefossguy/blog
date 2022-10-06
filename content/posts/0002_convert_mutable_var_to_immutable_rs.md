+++
draft = false
date = 2022-10-06T10:30:00+05:30
title = "How do immutable variables differ from constants in Rust"
description = "This is a series of posts that tries to solve various queries you might have while reading The Book."
slug = "immutable_vars_vs_constants_rs.md"
tags = ["the book", "rust", "rust variables", "rust constants"]
externalLink = ""
+++

## I have an mutable variable, how do I make it immutable?

Shadowing!

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

(The unused variable warning is snipped.)

`rustc` is complaining that on line 6, column 5 of "test.rs", it is referring
to `x` as immutable.

But didn't we declare `x` on as `mut` on line 2? Yes, we did.

Now, check the 5<sup>th</sup> line. There I assigned `x` to itself, using

---

<sub>Please excuse the absence of comments. I do not want to enable any tracking
(except for necessary embeds like YouTube and Twitter). I am not a web-dev and
have no way of manually verifying if a provider tracks you or not.</sub>
