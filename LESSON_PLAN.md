# Rust lesson plan

A path from "I can read basic Rust" to "I can write a real Rust program without fighting the compiler", for someone who already writes Go and Python. It is 13 modules and about 75 hours: roughly 15 weeks at 5 hours a week, or 8 weeks at 10.

Each module has reading, a project to build in this repo, and a "done when" check. The project matters more than the reading. If time is short, skim the reading and build the project.

## How to work through it

Create one crate per module with `cargo new m01-wc`, `cargo new m02-tokens`, and so on. Module 8 turns them into a single workspace.

Run `cargo clippy` on everything you write. Clippy is a linter that teaches idiomatic Rust one warning at a time, and it is the cheapest tutor available.

Read compiler errors fully, including the `help:` lines. Rust's errors usually contain the fix. `rustc --explain E0382` gives a longer explanation of any error code.

When the borrow checker blocks you in the first few modules, calling `.clone()` to get unstuck is fine. Note where you did it and come back after module 7.

The main text is [The Rust Programming Language](https://doc.rust-lang.org/book/) ("the Book"). Each module links straight to its chapters. [Rustlings](https://github.com/rust-lang/rustlings) provides small fix-the-code exercises grouped by topic, and [Rust by Example](https://doc.rust-lang.org/rust-by-example/) is useful when you want to see a feature in ten lines.

## Schedule

| # | Module | Hours | Project |
|---|--------|-------|---------|
| 0 | Setup | 0.5 | Hello world with tooling working |
| 1 | Syntax and basic types | 4 | `wc` clone |
| 2 | Ownership and borrowing | 6 | Tokenizer |
| 3 | Enums and pattern matching | 4 | RPN calculator |
| 4 | Error handling | 3 | Calculator with real errors |
| 5 | Collections, iterators, closures | 5 | Log analyzer |
| 6 | Traits and generics | 6 | Pluggable output formats |
| 7 | Lifetimes | 4 | Zero-copy parser |
| 8 | Modules, testing, tooling | 3 | Workspace with tests |
| 9 | Smart pointers and interior mutability | 5 | Tree and graph structures |
| 10 | Threads and shared state | 5 | Parallel file hasher |
| 11 | Async | 6 | Concurrent URL checker |
| 12 | Capstone | 20 | A tool you would actually use |

## Module 0: setup (30 minutes)

Install the rust-analyzer extension in your editor and turn on inlay hints, which show inferred types inline. Seeing the types the compiler inferred removes a lot of early confusion.

Build: `cargo new m00-hello`, then run `cargo run`, `cargo fmt`, `cargo clippy`, and `cargo test` once each so you know what they do.

Done when: your editor shows type hints and red squiggles for a deliberate type error.

## Module 1: syntax and basic types (4 hours)

Variables and `mut`, shadowing, integer and float types, tuples, arrays, structs, methods in `impl` blocks, functions, `if`, `loop`, `while`, `for`.

Coming from Go and Python, three things are new. Variables are immutable unless marked `mut`. Almost everything is an expression, so `if` and blocks return values and the last line of a function without a semicolon is its return value. There is no zero value and no `nil`: every variable must be initialized.

Read: the Book chapters [Programming a Guessing Game](https://doc.rust-lang.org/book/ch02-00-guessing-game-tutorial.html), [Common Programming Concepts](https://doc.rust-lang.org/book/ch03-00-common-programming-concepts.html), and [Using Structs to Structure Related Data](https://doc.rust-lang.org/book/ch05-00-structs.html). Rustlings: `variables`, `functions`, `if`, `primitive_types`, `structs`.

Build: a `wc` clone that takes a file path from `std::env::args` and prints line, word, and byte counts. Use `.unwrap()` on errors for now.

Done when: your output matches the real `wc` on three different files.

## Module 2: ownership and borrowing (6 hours)

This is the module that makes Rust Rust. Take it slowly and do not skip the exercises.

Every value has one owner, and the value is freed when the owner goes out of scope. Assigning or passing a value moves it unless the type is `Copy`. References borrow without taking ownership: any number of `&T` shared references, or exactly one `&mut T` exclusive reference, never both at once. `String` owns heap text and `&str` borrows a view of it. The same relationship holds for `Vec<T>` and `&[T]`.

Go and Python both hide this behind a garbage collector. The closest Go intuition is "a slice is a view into a backing array", except Rust checks at compile time that the backing array outlives the view and that nobody mutates it while you are looking.

Read: the Book chapter [Understanding Ownership](https://doc.rust-lang.org/book/ch04-00-understanding-ownership.html), twice if needed. Rustlings: `move_semantics`, `strings`, `vecs`.

Build: a tokenizer library with functions like `fn first_word(s: &str) -> &str` and `fn split_tokens(s: &str) -> Vec<&str>`, written by hand without `split_whitespace`. Then write `fn uppercase_in_place(s: &mut String)`.

Done when: you can predict, before compiling, whether a snippet fails with "use of moved value" or "cannot borrow as mutable", and explain why `&str` is the better parameter type than `&String`.

## Module 3: enums and pattern matching (4 hours)

Enums whose variants carry data, `match`, `if let`, `let else`, destructuring, and `Option<T>`.

Rust enums are sum types, which neither Go nor Python has natively. They replace Go's `iota` constants, interface-plus-type-switch patterns, and `nil` checks. `match` must be exhaustive, so adding a variant makes the compiler list every place you need to update.

Read: the Book chapters [Enums and Pattern Matching](https://doc.rust-lang.org/book/ch06-00-enums.html) and [Patterns and Matching](https://doc.rust-lang.org/book/ch19-00-patterns.html). Rustlings: `enums`, `options`.

Build: a reverse Polish notation calculator. Define `enum Token { Number(f64), Op(Operator) }`, parse input into `Vec<Token>` using your module 2 tokenizer, and evaluate with a stack. Return `Option<f64>`, with `None` for invalid input.

Done when: `3 4 + 2 *` prints `14`, and your code contains no `unwrap()` outside `main`.

## Module 4: error handling (3 hours)

`Result<T, E>`, the `?` operator, `panic!` versus returning an error, defining an error enum, implementing `From` for conversion, and the `thiserror` and `anyhow` crates.

`Result` is Go's `(value, err)` made into one type you cannot ignore. `?` is `if err != nil { return err }` in one character, and it converts error types through `From` on the way out. The convention: libraries define precise error enums (`thiserror` writes the boilerplate), applications use `anyhow` to attach context and bubble errors up to `main`.

Read: the Book chapter [Error Handling](https://doc.rust-lang.org/book/ch09-00-error-handling.html). Rustlings: `error_handling`.

Build: change the calculator to return `Result<f64, CalcError>` with variants such as `StackUnderflow`, `UnknownToken(String)`, and `DivideByZero`. Write the error type by hand first, including `Display`, then replace it with `thiserror`.

Done when: every bad input produces a specific message, and `main` returns `Result`.

## Module 5: collections, iterators, closures (5 hours)

`Vec`, `HashMap`, `HashSet`, the `Iterator` trait, adaptors (`map`, `filter`, `fold`, `zip`, `enumerate`, `flat_map`), `collect`, and closures with their three traits `Fn`, `FnMut`, and `FnOnce`.

Iterator chains are the idiomatic Rust loop. They read like Python generator expressions and compile to the same machine code as a hand-written loop. The distinction between `iter()` (borrows), `iter_mut()` (borrows mutably), and `into_iter()` (consumes) is ownership from module 2 showing up again.

Read: the Book chapters [Common Collections](https://doc.rust-lang.org/book/ch08-00-common-collections.html) and [Functional Language Features: Iterators and Closures](https://doc.rust-lang.org/book/ch13-00-functional-features.html). Rustlings: `hashmaps`, `iterators`.

Build: a log analyzer that reads a web server access log and reports the top 10 paths, the count per status code, and the busiest hour. Write it with `for` loops first, then rewrite each report as an iterator chain.

Done when: the iterator version has no `mut` variables outside the `HashMap` building step.

## Module 6: traits and generics (6 hours)

Defining and implementing traits, generic functions and structs, trait bounds and `where` clauses, `impl Trait`, trait objects with `dyn`, `#[derive]`, and the standard traits you will meet daily: `Debug`, `Display`, `Clone`, `PartialEq`, `Default`, `From` and `Into`, `Iterator`.

Traits are Go interfaces with two differences. Implementation is explicit (`impl Summary for Article`), not structural. And generics are monomorphized by default: the compiler generates a copy per concrete type, which is static dispatch with no runtime cost. `Box<dyn Trait>` gives you the Go-style runtime dispatch when you need a mixed collection.

Read: the Book chapter [Generic Types, Traits, and Lifetimes](https://doc.rust-lang.org/book/ch10-00-generics.html) (stop before lifetimes) and the section [Using Trait Objects to Abstract over Shared Behavior](https://doc.rust-lang.org/book/ch18-02-trait-objects.html). Rustlings: `generics`, `traits`.

Build: give the log analyzer a `trait Formatter` with text, JSON, and CSV implementations, selected by a command-line flag. Do it once with a generic `fn report<F: Formatter>` and once with `Box<dyn Formatter>`. Then implement `Iterator` for a struct of your own, such as a Fibonacci sequence.

Done when: you can say when you would pick generics over `dyn`, and why a `Vec` of mixed formatters needs `Box`.

## Module 7: lifetimes (4 hours)

Lifetime annotations on functions and structs, the elision rules that let you omit them most of the time, and `'static`.

A lifetime annotation does not change how long anything lives. It describes a relationship the compiler then checks: "the returned reference is valid no longer than this input". You have been relying on them since module 2, and the compiler was filling them in.

Read: the Book section [Validating References with Lifetimes](https://doc.rust-lang.org/book/ch10-03-lifetime-syntax.html). Rustlings: `lifetimes`.

Build: a parser for `key=value` config files whose output struct holds `&str` slices into the input, with no `String` allocations: `struct Config<'a> { entries: Vec<(&'a str, &'a str)> }`. Then revisit the `.clone()` calls you noted in earlier modules and remove the ones you can.

Done when: you can explain the error you get from returning a reference to a local variable, and why `Config` cannot outlive the input string.

## Module 8: modules, testing, tooling (3 hours)

`mod`, `pub`, `use`, splitting a crate across files, library plus binary crates, workspaces, unit tests in a `#[cfg(test)]` module, integration tests in `tests/`, doc tests, and adding dependencies.

Privacy is per module and private by default, where Go uses capitalization per package. Unit tests live in the same file as the code and can test private functions.

Read: the Book chapters [Packages, Crates, and Modules](https://doc.rust-lang.org/book/ch07-00-managing-growing-projects-with-packages-crates-and-modules.html) and [Writing Automated Tests](https://doc.rust-lang.org/book/ch11-00-testing.html), and the section [Cargo Workspaces](https://doc.rust-lang.org/book/ch14-03-cargo-workspaces.html). Rustlings: `modules`, `tests`.

Build: convert this repo into one Cargo workspace. Split the calculator into a library crate and a thin binary crate. Add unit tests, one integration test, and one doc test. Replace hand-rolled argument parsing with `clap`.

Done when: `cargo test --workspace` and `cargo clippy --workspace` both pass clean from the repo root.

## Module 9: smart pointers and interior mutability (5 hours)

`Box<T>` for heap allocation and recursive types, `Rc<T>` for shared ownership, `RefCell<T>` for mutation checked at runtime, `Weak<T>` for breaking cycles, and the `Deref` and `Drop` traits.

This is where designs that are trivial with a garbage collector (trees with parent pointers, graphs, doubly linked lists) get hard. The lesson is partly the tools and partly learning to restructure: storing nodes in a `Vec` and referring to them by index is often better than `Rc<RefCell<Node>>`.

Read: the Book chapter [Smart Pointers](https://doc.rust-lang.org/book/ch15-00-smart-pointers.html), then the first three chapters of [Learn Rust With Entirely Too Many Linked Lists](https://rust-unofficial.github.io/too-many-lists/).

Build: a binary search tree with `Option<Box<Node>>` supporting insert, lookup, and in-order iteration. Then a small directed graph twice: once with `Rc<RefCell<Node>>`, once with a `Vec<Node>` and index-based edges.

Done when: you can explain why the tree needs `Box`, and which graph version you would ship.

## Module 10: threads and shared state (5 hours)

`std::thread`, scoped threads, `move` closures, channels with `mpsc`, `Arc<Mutex<T>>`, `RwLock`, atomics, the `Send` and `Sync` marker traits, and the `rayon` crate.

Go says "share memory by communicating" and trusts you to follow it. Rust lets you do either and checks both at compile time: a data race is a compile error. A `Mutex` owns the data it protects, so you cannot touch the data without holding the lock.

Read: the Book chapter [Fearless Concurrency](https://doc.rust-lang.org/book/ch16-00-concurrency.html). Rustlings: `threads`.

Build: a tool that walks a directory and prints a SHA-256 hash per file. Write it three ways: worker threads fed by a channel, threads sharing an `Arc<Mutex<Vec<_>>>` of results, and `rayon`'s `par_iter`.

Done when: all three versions produce the same output, and you have read and understood one "cannot be sent between threads safely" error.

## Module 11: async (6 hours)

`async fn`, `.await`, futures as lazy state machines, the Tokio runtime, `tokio::spawn`, `join!` and `select!`, async channels, timeouts, and cancellation.

Goroutines are preemptive, cheap threads managed by a runtime built into the language. Rust async is different in kind: an `async fn` returns a future that does nothing until awaited, the runtime is a library you choose, and tasks only yield at `.await`. Calling blocking code inside a task stalls the executor thread. `select!` plays the role of Go's `select`.

Read: the Book chapter [Fundamentals of Asynchronous Programming](https://doc.rust-lang.org/book/ch17-00-async-await.html), then the [Tokio tutorial](https://tokio.rs/tokio/tutorial) through "Select".

Build: a URL checker that reads URLs from a file, fetches them concurrently with `reqwest`, limits concurrency to N with a `Semaphore`, applies a per-request timeout, and prints status and latency.

Done when: 100 URLs with a limit of 10 finish in about a tenth of the sequential time, and Ctrl-C shuts down cleanly.

## Module 12: capstone (20 hours)

Pick something you would actually use, ideally a tool you have already written in Go or Python so the design is known and only the Rust is new. Good shapes are a CLI that talks to an HTTP API, or a small HTTP service with `axum` and a SQLite database through `sqlx`.

Requirements: `clap` for arguments, `serde` for serialization, a `thiserror` error type in the library and `anyhow` in the binary, `tracing` for logs, tests for the core logic, and zero clippy warnings.

Done when: you use it for a week without needing to fix it.

## After the plan

Read [Rust for Rustaceans](https://rust-for-rustaceans.com/) by Jon Gjengset for the intermediate material this plan skips: macros, `unsafe`, FFI, and API design. Read the source of a crate you used in the capstone. Contribute a small fix to one.
