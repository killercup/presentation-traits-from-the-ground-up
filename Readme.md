---
title: "Traits from the Ground Up"
author: Pascal Hertleif
date: 2018-09-05
categories:
- rust
- presentation
theme: pascal
progress: true
slideNumber: true
history: true
---
## Hi, I'm Pascal Hertleif

- Web dev & Rust
- Co-organizer of [Rust Cologne]
- {[twitter],[github]}.com/killercup
- Rust-centric blog: [deterministic.space]

[Rust Cologne]: http://rust.cologne/
[twitter]: https://twitter.com/killercup
[github]: https://github.com/killercup
[deterministic.space]: https://deterministic.space/

::: notes

- I've been working with Rust since early 2014
- I have stickers!

:::

- - -

**Warning:** Lots of code

. . .

Interrupt me when you have a question

. . .

(Please use the mic)

# Data and Behavior

## Let's do stuff

```rust
fn make_true(input: &str) -> String {
  format!("{}!!", input)
}
```

::: notes

- Takes a reference string slice (`&str`)
- returns a new (true) `String`

:::

## Wrap data

```rust
struct Fact { text: String }

fn make_true(input: &Fact) -> Fact {
  Fact { text: format!("{}!!", input.text) }
}
```

# Add Behavior to Data

## How to write a method

> 1. Write: `impl YourType {`
> 2. Write: some function
> 3. Write: `}`

## `make_true` as a method

```rust
struct Fact { text: String }

impl Fact {
  fn make_true(&self) -> Fact {
    Fact { text: format!("{}!!", self.text) }
  }
}
```

## Using the method

```rust
let fact = Fact { text: String::from("Lorem ipsum") };
let true_fact = fact.make_true();
```

## self?

Methods can operate on the data

Access the data using a form of `self` as first parameter

::: notes

- A form of `self` as first parameter makes a function an instance method

:::

## "A form of `self`"?

You need to specify how you want `self`:

- `&self`: Borrowed, read-only version
- `&mut self`: Mutable, borrowed version
- [`mut`] `self`: Owned version, do whatever you want with it (incl. destroying it)

::: notes

Basically the same `borrowck` rules as always

:::

## Another `make_true`

```rust
struct Fact { text: String }

impl Fact {
  fn make_true(&mut self) {
    self.text.push_str("!!");
  }
}
```

## Or:

```rust
struct Fact { text: String }

impl Fact {
  fn make_true(mut self) -> Fact {
    self.text.push_str("!!");
    self
  }
}
```

::: notes

You can no longer use the original fact

:::

# Is this a trait?

::: notes

Yes it is, it just doesn't have a name

:::

## Let's give it a name

```rust
trait Truth {
  fn make_true(&self) -> Self
}
```

::: notes

Let's use the `&self` version because
we just need to look at a fact
to create a new alternative fact

:::

## Now we can implement it

```rust
impl Truth for Fact {
  fn make_true(&self) -> Self {
    Fact { text: format!("{}!!", self.text) }
  }
}
```

::: notes

This is the magic sauce right here

:::

## Implement it for everything!

```rust
impl Truth for i32 {
  fn make_true(&self) -> Self {
    42
  }
}
```

# Abstract over behavior

## `T: Truth`

```rust
fn print_news<T: Truth>(facts: &[T]) {
  for fact in facts {
    let fact = fact.make_true();
    println!("{}", fact);
  }
}
```

- - -

Wait a second:

```plain
error[E0277]: `T` doesn't implement `std::fmt::Display`
  --> src/lib.rs:16:20
   |
16 |     println!("{}", fact.make_true());
   |                    ^^^^^^^^^^^^^^^^ `T` cannot be formatted with the default formatter
   |
   = help: the trait `std::fmt::Display` is not implemented for `T`
   = note: in format strings you may be able to use `{:?}` (or {:#?} for pretty-print) instead
   = help: consider adding a `where T: std::fmt::Display` bound
   = note: required by `std::fmt::Display::fmt`
```

## What do we know about `T`?

> - It implements `Truth`
> - ???
> - Nope, that's it*

::: notes

Auto traits are a thing: Sized, Sync, Send

:::

- - -

```rust
use std::fmt;

fn print_news<T: Truth + fmt::Display>(facts: &[T]) {
  for fact in facts {
    let fact = fact.make_true();
    println!("{}", fact);
  }
}
```

- - -

But we'll also need:

```rust
impl fmt::Display for Fact {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "{}", self.text)
    }
}
```

- - -

Here we go:

```rust
fn main() {
    let facts = vec![
      Fact { text: String::from("lorem ipsum") },
    ];

    print_news(&facts);
}
```

```plain
$ cargo run
lorem ipsum!!
```

## `where` syntax

Instead of

```rust
fn print_news<T: Truth + fmt::Display>(facts: &[T]) {}
```

we can also write

```rust
fn print_news<T>(facts: &[T]) where T: Truth + fmt::Display {}
```

## Using Generics in `impl`s

```rust
impl<T: Debug> Foo for Vec<T> {
  // ...
}
```

# Associated items

## Associated functions

That's what we've been writing all this time

## Associated types

specify types you need to refer to that are specific to each `impl` block

```rust
trait Iterator {
  type Item;
  fn next(&mut self) -> Option<Self::Item>;
}
```

```rust
impl Iterator for FourIntegers {
  type Item = i32;
  fn next(&mut self) -> Option<i32> {}
}
```

## Associated constants

```rust
trait SuperLog {
  const LABEL: Display;

  fn log(&self, f: &mut fmt::Formatter) -> fmt::Result;
}
```

# Good to know

## Unambiguous function call syntax

`fact.make_true()`

expands to

`<Fact as Truth>::make_true(fact)`

## Operators are traits

`a + b` expands to `std::ops::Add::add(a, b)`

because of:

```rust
impl Add<i32> for i32 {
  type Output = i32;
  // ...
}
```

## Marker traits

```rust
trait Fancy {}

impl Fancy for Vec<String> {}
```

# Thanks!

- - -

## Any questions?

I am Pascal

{[twitter],[github]}.com/killercup

Slides: [git.io/traits-from-the-ground-up](https://git.io/traits-from-the-ground-up)
