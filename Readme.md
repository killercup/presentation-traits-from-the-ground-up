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
    Fact { text: format!("{}!!", input.text) }
  }
}
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
- `self`: Owned version, do whatever you want with it (incl. destroying it)

::: notes

Basically the same `borrowck` rules as always

:::

## Another `make_true`

```rust
struct Fact { text: String }

impl Fact {
  fn make_true(&mut self) {
    input.text.push_str("!!");
  }
}
```

## Or:

```rust
struct Fact { text: String }

impl Fact {
  fn make_true(self) -> Self {
    input.text.push_str("!!");
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
  fn make_true(self) -> Self;
}
```

## Now we can implement it

```rust
impl Truth for Fact {
  fn make_true(self) -> Self {
    input.text.push_str("!!");
    self
  }
}
```

::: notes

This is the magic sauce right here

:::
