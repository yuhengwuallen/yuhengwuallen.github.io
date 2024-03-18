---
title: "the way to master rust"
date: 2024-03-13
# weight: 1
# aliases: ["/first"]
tags: ["rust", "course"]
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

useful links:

1. [the book](https://doc.rust-lang.org/book/)
2. [rust by examples](https://doc.rust-lang.org/rust-by-example/)
3. [Rustonomicon](https://doc.rust-lang.org/nomicon/intro.html)
4. [rust RFC](https://rust-lang.github.io/rfcs/)

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


### Interior Mutability

Interior mutability in Rust is a design pattern that allows you to modify data even when there are immutable references to that data, thus bypassing Rust's usual borrowing rules that typically disallow modifying data through an immutable reference. This is achieved by using certain types provided by the Rust standard library, which use unsafe code inside to safely encapsulate and manage mutations. The primary purpose of this pattern is to enable the modification of data in a controlled manner without violating the safety guarantees of the Rust language.

Read this link thoroughly: [https://doc.rust-lang.org/std/cell/index.html](https://doc.rust-lang.org/std/cell/index.html). For fully understanding, please read this doc carefully. The contents as follows are mainly directly cv from there.

#### Single Thread version
reference to: [https://doc.rust-lang.org/std/cell/](https://doc.rust-lang.org/std/cell/)
##### `Cell<T>`
`Cell` implements the interior mutability by moving values in and out of the cell. And is usually for simple types where copying or moving values isn't too resource intensive.

##### `RefCell<T>`
`RefCell` uses Rust's lifetimes to implement "dynamic borrowing" [figure out how it achieves this goal in the source code later], a process whereby one can claim temporary, exclusive, mutable access to the inner value. This _borrows_ are checked at _run time_ and is different from native reference types which are tracked at _compile time_.   
An immutable reference to a RefCell’s inner value (&T) can be obtained with borrow, and a mutable borrow (&mut T) can be obtained with borrow_mut. When these functions are called, they first verify that Rust’s borrow rules will be satisfied: any number of immutable borrows are allowed or a single mutable borrow is allowed, but never both. If a borrow is attempted that would violate these rules, the thread will panic.

For types that implement `Copy`, the get method retrieves the current interior value by duplicating it.
For types that implement `Default`, the take method replaces the current interior value with `Default::default()` and returns the replaced value.   

All types have:    
`replace`: replaces the current interior value and returns the replaced value.
`into_inner`: this method consumes the `Cell<T>` and returns the interior value.
`set`: this method replaces the interior value, dropping the replaced value.

##### `OnceCell<T>`
`OnceCell<T>` is somewhat of a hybrid of Cell and RefCell that works for values that typically only need to be set once. This means that a reference &T can be obtained without moving or copying the inner value (unlike Cell) but also without runtime checks (unlike RefCell). However, its value can also not be updated once set unless you have a mutable reference to the OnceCell.

`OnceCell` provides the following methods:    
`get`: obtain a reference to the inner value
`set`: set the inner value if it is unset (returns a Result)
`get_or_init`: return the inner value, initializing it if needed
`get_mut`: provide a mutable reference to the inner value, only available if you have a mutable reference to the cell itself.
The corresponding Sync version of `OnceCell<T>` is `OnceLock<T>`.

#### `Mutex<T>`

#### `RwLock<T>`