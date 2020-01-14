+++
title = "Debug for Fn, FnOnce, FnMut"
description = "Rust: How to impl Debug for Fn, FnOnce, FnMut"
author = "Boyd Johnson"
date = 2020-01-14

[taxonomies]
categories = ["Rust"]
tags = ["Rust", "tips"]
+++

A good rule in Rust is that every struct should have `std::debug::Debug`. But what if that struct contains closures.

I initially had a struct that looked like:

```rust
pub struct Move {
    from: CardIndex,
    to: CardIndex,
    on_after: Option<DisplayFn>,
    on_after_reverse: Option<DisplayFn>,
}

pub type DisplayFn = Box<dyn FnOnce(&mut Card)>;
```

The usecase for this code is a cardgame that has an undo button. When a card moves, its display may change, but when undo happens its display needs to be reversed.

If we try to derive Debug for it, we will get a compiler error because FnOnce does not impl Debug.

The solution is to declare a trait that has FnOnce and impl it for all functions that have FnOnce, and then impl Debug for that trait.

```rust
pub trait DisplayFnT: FnOnce(&mut Card);

impl<F> DisplayFnT for F where F: FnOnce(&mut Card) { }

impl std::fmt::Debug for DisplayFnT {
    fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
        write!(f, "DisplayFunction")
    }
}

pub type DisplayFn = Box<dyn DisplayFnT>;
```

Then the final struct looks like:

```rust
#[derive(Debug)]
pub struct Move {
    from: CardIndex,
    to: CardIndex,
    on_after: Option<DisplayFn>,
    on_after_reverse: Option<DisplayFn>,
}

```

I got the solution to this problem from [this users.rust-lang.org question and answer.](https://users.rust-lang.org/t/is-it-possible-to-implement-debug-for-fn-type/14824)
