# Anatomy

We’ve got a few tools to get to know
* rustup - manage tools and versions of toolchains
* rustc - rust compiler
* cargo - manage modules locally and remotely and drives rustc

**EXERCISE: Open a terminal and create a new package with `cargo new training` and go to that directory with `cd training`**

Now we have a Cargo.toml which defines our project, not unlike a package.json if you're familiar with Node.js, it defines dependencies we're using and other project information:
```toml
[package]
name = "training"
version = "0.1.0"
authors = ["First Last"]
edition = "2018"

[dependencies]
```
In src folder we have main.rs, a Rust file. In this case it generated a simple hello world. 
```rust
fn main() {
    println!("Hello, world!");
}
```

We're starting to get some syntax for you. Notice functions are denoted `fn`, we use semicolons to end expressions, and the exclamation after `println!()` means that is a function-like macro. We'll talk more about [macros](https://doc.rust-lang.org/book/ch19-06-macros.html) later.

Generally we'll interact with the compiler via Cargo. Cargo drives the rustc compiler and linker all under the hood. We can `cargo build` or better yet `cargo run` and save ourselves a step:
```bash
$ cargo run
   Compiling training v0.1.0 (/Users/firstlast/training)
    Finished dev [unoptimized + debuginfo] target(s) in 0.50s
     Running `/Users/firstlast/.cache/target/debug/training`
Hello, world!
```

The default build directory is target, and by default we got a debug build
```bash
$ ls target/debug/
build		examples	native		training.d
deps		incremental	training	training.dSYM
```

Note, we could run or debug that built asset directly:
```bash
./target/debug/training
Hello, world!
```

Also note, we could have compiled this simple file with the rustc compiler directly
```bash
$ rustc src/main.rs
$ ./main
Hello, world!
```
However in practice almost no projects are single files require merging multiple modules from within our project and without and thus Cargo is THE way we interact with Rust.
