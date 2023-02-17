+++
draft = false
date = 2022-10-06T10:30:00+05:30
title = "What are the differences between Constants and Variables in Rust?"
description = "This is a series of posts that tries to solve various queries you might have while reading The Book."
slug = "immutable-vars-vs-constants-rs.md"
tags = ["the book", "rust", "rust variables", "rust constants"]
externalLink = ""
+++

The reason why the Rust language's developers can advertise features like
_thread safety_ and _memory safety guarantee_ is because of a fundamental
design ideology of immutable variables. Immutable variables are the type of
variables where, once you assign a value, it does not change.

Rust also has constants. So you might wonder "Why does Rust have Immutable
variables _and_ Constants? Aren't they the same thing?"

In this blog post, I will explain the basics of variables and constants
in Rust, and how an immutable variable differs from a mutable variable.

## Immutable variables VS Constants

If, like me, Rust is not your first programming language, this confusion is
bound to occur sooner rather than later--"Why do either immutable variables
or constants exist in Rust?"

If you have written a simple program in Rust, you will realize that Rust has
2 types of variables. One is the type which allows changing a value, even
after it is assigned, called _mutable variables_. The second type is the one
in question, immutable variables. Unless explicitly specified, Rust assumes
a variable is immutable. Meaning, once a value is assigned to your variable,
that value will never be allowed to be changed.

Rust only has one type for constants. The one which makes sense. Immutable by
default. Once a value is assigned, it is not allowed to change.

## Type inferencing

When you declare a variable in Rust, you do it using the `let` keyword. This
is different than other low-level languages like C and C++, where you are to
explicitly specify the data-type of a variable using the appropriate keyword
like, `int`, `float`, `char`, etc.

Using the `let` keyword is necessary, but specifying a variable's data-type
is not necessary. You can either leave it to the Rust compiler (`rustc`) to
take a guess--which hardly misfires--or you can annotate the type yourself.

This guessing that `rustc` does is called "type inferencing".

Type inferencing is absent for constants. It is the job of a programmer to
provide a type for a constant that is declared.

Take a look at the following code:

Filename: const_example.rs

```rust
fn main() {
    let my_var = -128;
    const MY_CONST = -128;
}
```

In here, I am declaring an immutable variable (`my_var`) with the value "-128"
and a constant (`MY_CONST`) with the same value of "-128". Another similarity
is that neither of them are type annotated.

Let's try and compile this.

```bash
$ rustc const_example.rs
error: missing type for `const` item
 --> const_example.rs:3:11
  |
3 |     const MY_CONST = -128;
  |           ^^^^^^^^ help: provide a type for the constant: `MY_CONST: i32`

error: aborting due to previous error
```

Hmm...

`rustc` is complaining about an error on line 3 (where we declared our
constant). The highlighted part is the name (`MY_CONST`).

The help states "provide a type for the constant". The help message also
included a "recommended/suggested type" (`i32`) but it was not applied.

Hence, one of the difference between an immutable variable and a constant in
Rust is that constants **need** type annotation. Type inferencing is **not**
applicable to constants.

## Scope of declaration

Another point of difference between an immutable variable and a constant in
Rust is its scope of declaration.

Constants can be declared globally (before/outside the `main` function).
Variables, immutable or otherwise, cannot be declared globally.

Let's see this with an example.

Filename: const_example.rs

```rust
const MY_CONST:i32 = -128;
let my_var: i32 = -128;

fn main() {
    println!("{my_var} {MY_CONST}");
}
```

As you can see here, I have mostly the same code as above, but I have moved
both, the variable and the constant declaration, in the global scope.

'tis compile time!

```bash
$ rustc const_example.rs
error: expected item, found keyword `let`
 --> const_example.rs:2:1
  |
2 | let my_var: i32 = -128;
  | ^^^ expected item

error: aborting due to previous error
```

An error :(

The way I wrote the code should give you a hint. We have an error on the
2<sup>nd</sup> line. Our constant is declared on the 1<sup>st</sup> line.

This means, `rustc` did not see any problems with a constant in the global
scope. But it does have a problem with our immutable variable if it is
declared in the global scope.

## Assignable values

Another major difference between variables (immutable or otherwise) and
constants in Rust is that a constant **cannot** have a value that can be
calculated **only at run-time**.

What do I mean by this?

Take a look at the following code:

Filename: const_example.rs

```rust
cat const_example.rs
fn main() {
    let pi: f32 = 3.14;
    const PI_TIMES_TWO: f32 = pi * 2;
}
```

In this code, I am declaring an immutable variable (`pi`) and assigning it the
value of "3.14". Next, I declare a constant, with the same type as `pi`,
and I assign it the value of `pi * 2`.

Shall we compile?

```bash
$ rustc const_example.rs
error[E0435]: attempt to use a non-constant value in a constant
 --> const_example.rs:3:31
  |
3 |     const PI_TIMES_TWO: f32 = pi * 2;
  |     ------------------        ^^ non-constant value
  |     |
  |     help: consider using `let` instead of `const`: `let PI_TIMES_TWO`

error: aborting due to previous error

For more information about this error, try `rustc --explain E0435`.
```

As you can see, even though `pi` is an immutable variable, we get this error.
This is because `PI_TIMES_TWO` is dependent on the value stored in `pi`
to determine its own value. This is problematic because the values of
variables are **not** evaluated at compile-time.

The value assigned to a constant **must not** be calculated/evaluated at
_run-time_.

## Compile-time vs Run-time

As I just proved, constants can not be assigned a value that will be
calculated at run-time. That must raise a question if the core reason for the
existence of immutable variables and constants must be related to the
differences in run-time and compile-time.

While I am **not** someone who has contributed to the design of the Rust
language, I am inclined to assume that this might be the reason.

You can use the value assigned to a constant during compile-time, to make
decisions _while_ the code is being compiled. This can not be done using
variables, immutable or otherwise.

Let me demonstrate this using an example.

Filename: const_example.rs

```rust
fn main() {
    let arr_len_var: usize = 5;
    const ARR_LEN_CONST: usize = 5;

    let arr_from_const: [i32; ARR_LEN_CONST];
    let arr_from_var: [i32; arr_len_var];
}
```

In this example, I am doing the following:

 1. Declare an immutable variable with the value "5".
 2. Declare a constant with the value "5".
 3. Create an empty array, using the value of an immutable variable as the
 "array size/length".
 4. Create another empty array, using the value of a constant as the "array
 size/length"

If, the jibberish that I just wrote above is correct, we should expect a
compilation error on the 6<sup>th</sup> line.

```bash
$ rustc const_example.rs
error[E0435]: attempt to use a non-constant value in a constant
 --> const_example.rs:6:29
  |
2 |     let arr_len_var: usize = 5;
  |     --------------- help: consider using `const` instead of `let`: `const arr_len_var`
...
6 |     let arr_from_var: [i32; arr_len_var];
  |                             ^^^^^^^^^^^ non-constant value

error: aborting due to previous error

For more information about this error, try `rustc --explain E0435`.
```

As expected.

`rustc` has 2 messages for us. The first message--after looking at the
6<sup>th</sup> line--is a suggestion for us; to change `arr_len_var` from
a variable to a constant.

The second message is an error message. It is complaining that the value
which determines the length of an array, is a "non-constant value".

"But I thought the values of immutable variables could never change?! Does
that not equate to a _non-constant value_?"

You are correct, but this is something different. You see--this is where I am
applying my knowledge of what I learnt about compiler design from my
college--constants are also _evaluated_ at compile-time.

This means, the following line -

```rust
    const PI: f32 = 3.14;
    println!("{PI}");
```

gets replaced with the following in the first few passes of the Rust compiler.

```rust
    const PI: f32 = 3.14;
    println!("3.14");
```

The constant got evaluated (expanded), at compile-time. This will not be be
the case for a variable, immutable or otherwise.

This is also what happens when we use a constant to determine the length/size
of an array.

## Shadows

You might know about shadowing in Rust. It refers to the act of referring
to a different storage/address using the same name.

Below is an example of shadowing:

```rust
let five = 5;
let five = 6;
```

Here, on the first line, we are declaring an immutable variable `five`. It
assigned the value "5". In the immediate next line, we declare the variable
`five` again. This time, we assign it "6".

What this does is, when `five` was first declared and assigned the value "5",
it was given a memory address to store that "5" which we assigned to it.
Assume this memory address to be `0x01`.

Then, when we declared `five` again; this time with a different value, "6";
a different memory address was given to our variable `five`. This, new memory
address stored the value "6".  Assume this memory address to be `0x02`.

Now, the older memory address (`0x01`) is not overwritten with "6", instead of
"5". It is still kept--maybe because this shadow was a local change and we
will need the previous value again? who knows--intact. But now, when we ask,
"Hey `five`, what is your assigned value?", it will check the memory location
`0x02` and give us a value from there; which is "6".

This isn't possible for constants.

 - You can not shadow a constant with a constant (of any type).
 - You can not shadow a constant with a variable (of any type).
 - You can not shadow a variable with a constant (of any type).

## Minor nit-picks

A few minor differences between a constant and an immutable variable are as
follows:

 - Variables are immutable by default, but they can also be mutable, if
 asked nicely. On the contrary, constants are _always_ immutable. (You cannot
 use the keyword `mut` next to the `const` keyword.)
 - To declare a constant, we use the `const` keyword, but to declare an
 immutable variable, we use the `let` keyword. A variable can be made
 immutable if, at the time of declaration, the `mut` keyword is used alongside
 the `let` keyword.

## Conclusion

The intelligent mind who were designing the Rust language were obviously not
out of their minds when they created variables that default to immutability
when constants would also exist.

To recap, constants need type annotations, but they can be declared in the
global scope; values assigned to constants cannot be something that is
calculated at run-time and they can not be shadowed.

---

<sub>Please excuse the absence of comments. I do not want to enable any tracking
(except for necessary embeds like YouTube and Twitter). I am not a web-dev and
have no way of manually verifying if a provider tracks you or not.</sub>
