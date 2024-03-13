---
title: "the way to master rust"
date: 2024-03-13
# weight: 1
# aliases: ["/first"]
tags: ["rust"]
author: "Yuheng"
showtoc: true
tocopen: true
draft: true
math: true
comments: false
description: "note & thinking of cs431 kaist-cp"
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: false
ShowWordCount: true
ShowRssButtonInSectionTermList: false
ShowShareButtons: false
---

## Rust

I use rust version 1.76.0 (07dca489a 2024-02-04).

### Ownership
The rules of `ownership`:

- Each value in Rust has an __owner__
- There can only be one owner at a time (may have shared ownership via Arc/Rc, invoking `clone` produces a new pointer to the same allocation in the heap)
- When the owner goes out of the scope, the value will be dropped

#### References & Borrowing
We call the action of creating a reference - borrowing, which won't take the ownership of the variable.

The rules of `References`:
- At any given time, you can have either one mutable reference or any number of immutable references.
- References must always be valid.

#### Lifetimes of References
In Rust, there is a `Borrow Checker` that compares scope of the references and owner to determine the validation of references.

For example, here we have a function which returns the longest string. In the longest(), we compare the length of the two references `x` and `y` and then return the reference of the longer one.

```rust
fn main() {
    let string1 = String::from("abcd");
    let string2 = "xyz";

    let result = longest(string1.as_str(), string2);
    println!("The longest string is {}", result);
}

fn longest(x: &str, y: &str) -> &str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

However, we will get error as follows:
```bash
$ cargo run
   Compiling chapter10 v0.1.0 (file:///projects/chapter10)
error[E0106]: missing lifetime specifier
 --> src/main.rs:9:33
  |
9 | fn longest(x: &str, y: &str) -> &str {
  |               ----     ----     ^ expected named lifetime parameter
  |
  = help: this function's return type contains a borrowed value, but the signature does not say whether it is borrowed from `x` or `y`
help: consider introducing a named lifetime parameter
  |
9 | fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
  |           ++++     ++          ++          ++

For more information about this error, try `rustc --explain E0106`.
error: could not compile `chapter10` due to previous error
```

Because when we pass the reference of the two references to the longest(), the owner of `x` and owner of `y` may vary in their lifetime. In a more special case, `x` may get out of the scope earlier than the `y`. If we get the shorter lifetime reference `y` in this circumstance, we then use the return reference in the scope of `x` after `y` is out of the scope, it will incur huge memory issue. Even though in current settings, the two variables owner have the same lifetime, the _compiler_ of Rust is not that smart to distinguish where the two have different lifetime. In this case, we must always annotate the lifetime to help the compiler borrow checker. The way in which we need to specify lifetime parameters depends on what our function is doing.

For detailed annotation on _struct_, _function_, _method_ and _generics_, check the link [https://doc.rust-lang.org/book/ch10-03-lifetime-syntax.html](https://doc.rust-lang.org/book/ch10-03-lifetime-syntax.html).

Currently(1.76.0), the compiler can auto analyze the lifetime based on three rules.
1. the compiler assigns a lifetime parameter to each parameter that’s a reference. `fn foo<'a, 'b>(x: &'a i32, y: &'b i32)`
2. if there is exactly one input lifetime parameter, that lifetime is assigned to all output lifetime parameters. `fn foo<'a>(x: &'a i32) -> &'a i32`
3. if there are multiple input lifetime parameters, but one of them is `&self` or `&mut self` because this is a method, the lifetime of `self` is assigned to all output lifetime parameters