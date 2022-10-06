+++
draft = false
date = 2022-10-06T10:30:00+05:30
title = "How do immutable variables differ from constants in Rust"
description = "This is a series of posts that tries to solve various queries you might have while reading The Book."
slug = "immutable_vars_vs_constants_rs.md"
tags = ["the book", "rust", "rust variables", "rust constants"]
externalLink = ""
+++

## Introduction

If you just started learning Rust, you might have noticed that its variables
are "immutable by default". Rust also has constants.

Doesn't this mean that Rust's "immutable variables" ≈ "constants"?

Not quite.

## How are immutable variables different than constants?

Only the property that "the value can never change" is true for both but that
is all that is common between Rust's immutable variables and constants.

Constants are used when you have to evaluate something at compile-time.
Meanwhile, immutable variables are used when you need a value at run-time.

Take a look at the following code snippet:

Filename: test.rs

```rust
fn main() {
    const ARRAY_LENGTH: usize = 5;
    let my_array: [usize; ARRAY_LENGTH];
}
```

(Populating the array is not necessary for our purposes.)

Let's compile it.

```bash
$ rustc test.rs
warning: unused variable: `my_array`
 --> test.rs:3:9
  |
3 |     let my_array: [usize; ARRAY_LENGTH];
  |         ^^^^^^^^ help: if this is intentional, prefix it with an underscore: `_my_array`
  |
  = note: `#[warn(unused_variables)]` on by default

warning: 1 warning emitted
```

A warning was generated, but not a compile-time error. So our code is "correct".
Since our code compiled, our constant, `ARRAY_LENGTH` was replaced with
the value "5" _while the code was being compiled_.
Now, let's add a few lines below it and comment out this one.

Filename: test.rs

```rust
fn main() {
    /*
    const ARRAY_LENGTH: usize = 5;
    let my_array: [usize; ARRAY_LENGTH];
    */

    let var_array_length: usize = 5;
    let new_array: [usize; var_array_length];
}
```

And, compile!

```bash
$ rustc test.rs
error[E0435]: attempt to use a non-constant value in a constant
 --> test.rs:8:28
  |
7 |     let var_array_length: usize = 5;
  |     -------------------- help: consider using `const` instead of `let`: `const var_array_length`
8 |     let new_array: [usize; var_array_length];
  |                            ^^^^^^^^^^^^^^^^ non-constant value

error: aborting due to previous error

For more information about this error, try `rustc --explain E0435`.
```

Oh no. Okay, let's do `rustc --explain E0435`.

```bash
$ rustc --explain E0435

    A non-constant value was used in a constant expression.

    Erroneous code example:

    ```
    let foo = 42;
    let a: [u8; foo]; // error: attempt to use a non-constant value in a constant
    ```

    'constant' means 'a compile-time value'.

    More details can be found in the [Variables and Mutability] section of the book.

    [Variables and Mutability]: https://doc.rust-lang.org/book/ch03-01-variables-and-mutability.html#differences-between-variables-and-constants

    To fix this error, please replace the value with a constant. Example:

    ```
    let a: [u8; 42]; // ok!
    ```

    Or:

    ```
    const FOO: usize = 42;
    let a: [u8; FOO]; // ok!
    ```
```

The most important part of this explainer is:

> **'constant' means 'a compile-time value'.**

This means, a constant is only used when you (technically `rustc`) are
compiling some code and want a change to occur _during compilation_.

All the references of a constant will be replaced by the value assigned to it
during compile time. You may `println!("{MY_CONSTANT}");`, but it will be changed
to `println!("5");` **during compilation** (assuming `MY_CONSTANT` is "5").

Another few _key_ differences between variables and constants as as follows:

 - Constants can't be shadowed _with the same type of variable_.
 - Constants can't be assigned a value that will be calculated at run-time.

## Conclusion

So, in conclusion, though `const` and `let` (without `mut`) are the same
in the sense that "the assigned value never changes", they are _used_ for
different scenarios. Constants are used when you want a value to determine
something **at compile-time** (like defining the length of an array). If you
want a value to be used **at run-time**, that is where immutable variables
become useful.

---

<sub>Please excuse the absence of comments. I do not want to enable any tracking
(except for necessary embeds like YouTube and Twitter). I am not a web-dev and
have no way of manually verifying if a provider tracks you or not.</sub>
