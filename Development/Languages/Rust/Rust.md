---
aliases:
  - Rust
tags:
  - learning
  - dev/lang/rust
date: 2026-05-12
---
**Sources**: [Rust](https://rust-lang.org/es/), [learn](https://rust-lang.org/es/learn/)

**Related:** [[Development]], [[C]], [[C++]]

---

## Description

_Rust_ is a multi-paradigm, systems programming language designed for **performance**, **safety**, and **reliability**. It is often compared to ``C`` and ``C++`` because it provides **low-level control over hardware and memory without the overhead of a garbage collector**.

---

## Key concepts

- **Memory Safety:** _Rust_ uses a unique **ownership system** with a "borrow checker" to manage memory. This allows the compiler to guarantee memory safety at compile time, preventing common bugs like null pointer dereferences and buffer overflows.

- **Performance:** It is a **compiled language** that produces highly optimized machine code, making it suitable for performance-critical applications like game engines, operating systems, and browser components.

- **Fearless Concurrency:** The same ownership rules that ensure memory safety also prevent "data races," making it much easier and safer to write programs that run multiple tasks simultaneously.

- **Zero-Cost Abstractions:** High-level features like generics and collections are designed to have no runtime performance cost, meaning you don't pay for what you don't use

___

## Examples

### Hello World!

```rust title:main.rs
fn main() {
	println!("Hello, world!");
}

```

---

## Claude Sessions