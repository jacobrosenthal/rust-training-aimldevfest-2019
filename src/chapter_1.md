# Chapter 1



## define your terms

Rust is a safe, typed, compiled, general programming language.
* Safe is a very overloaded term in Rust, but by default Rust uses static analysis at compile time to enforces rules about memory usage. Traditionally, managing your own memory in languages like C++ and Objective-C has been very tedious and still error prone. Rust's solution to this, The borrow checker, is probably the defining feature of the Rust language. You have to spend a little more time annotating your code to say who owns a variable at any given time with the borrow checker keeping you honest the whole time. It's like pair programming with a friend.
* Typed - As opposed to languages like javascript, Rust forces you to state the types of variables going into and out of functions. This helps you organize your intentions and acts much like a set of tests to make sure your code makes sense.
* Compiled - Rust has to do all its work up front at compile time and turns into a binary immediately. Compiles can be slow sometimes, but our code runs fast, and anywhere, as a result. [obligatory xkcd](https://xkcd.com/303/) 
* General. Much like most modern languages these days it's not strictly functional or object oriented (OO). Further it can be deployed almost anywhere. We can write backend server code, embedded microcontroller applications, and with WASM, even front end web applications, cloud functions and blockchains.



## anatomy of Rust program and some workflow

Weve got a few tools to get to know
* rustup - manage tools and versions of toolchains
* rustc - rust compiler
* cargo - manage modules locally and remotely and drives rustc

Open a terminal and create a new package with `cargo new training` and go to that directory with `cd training`

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

We're starting to get some syntax for you. Notice functions are denoted fn, we use semicolons to end expressions, and the exclamation after println! means that is a function-like macro. We'll talk more about macros](https://doc.rust-lang.org/book/ch19-06-macros.html) later. 

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

## general syntax

Weve got all the datatypes you would expect but you might want to glance through the Rust book [chapter on variables, functions, and control flow](https://doc.rust-lang.org/book/ch03-01-variables-and-mutability.html) just to update your mental models to Rust notation

We have signed and unsigned scalar types like u32 and i32 and we've got Strings. Variables are instantiated with let syntax, and notably are immutable by default.

The top of the [Rust standard library page](https://doc.rust-lang.org/std/) has a search box. Entering String there we find [std::string::String](https://doc.rust-lang.org/std/string/struct.String.html#method.from) with a bunch of example usage right there for us. You can also click the src button and be taken straight to the Rust implementation and you can even edit those examples and run them right in your browser to confirm your understanding.

> While you totally can thrash around on stack overflow, and we all do, there really is an authoratative source that you should check first.

Lets steal an example:
```rust
let first = String::from("first");
```

First, note we dont need to import anything (we call it `use`) to use this type. A portion of the standard library is in our namespace automatically, which we call the [prelude](https://doc.rust-lang.org/std/prelude/index.html). Basically Rust puts use `std::prelude::v1::*;` at the top of your file and you get access to those members. By no means is everything in there, but a lot is, which is what kept you from explicitly writing `use std::string::String` at the top of your file in this case.

Also notice we didn't have to explicitly type our variable named first. What Rust *can* figure it out, *it will* and so its entirely idiomatic to omit type annotations. However if you or the compiler are having trouble or getting odd type errors, start annotating some of your types like to see if you can give the compiler a hand. Its also a great way to figure out what type you actually have in case you're not sure, let the compiler (or linter) tell you.

```rust
let last:String = String::from("last");
```

You know where were going, printing your name. In Rust our printf replacement character is `{}`. Following the println! documentation down the rabbit hole will send us to the [formatters section](https://doc.rust-lang.org/std/fmt/index.html) page and we find all the formatters which you would expect like hex `{:x}`, binary `{:b}`, etc. We're going to focus on the 'empty' Display formatter `{}` for now which is a kind of a pretty printer in Rust. As long as whover wrote our type implemented the Display pretty printer trait this will work great (cue ominous music).
```rust
fn main() {
	let first = String::from("first");
	let last = String::from("last");
    println!("Hello, {} {}", first, last);
}
```
String prety printing results in rather clean output in this case:
```bash
$ cargo run
   Compiling training v0.1.0 (/Users/firstlast/Downloads/training)
    Finished dev [unoptimized + debuginfo] target(s) in 0.77s
     Running `target/debug/training`
Hello, first last
```

Objects, we call them structs, should be very familiar. You can define a new struct in any scope you like and we can name and type their members.
```rust
struct Name {
    first: String,
    last: String,
}

fn main() {
    let first = String::from("first");
    let last = String::from("last");

    let name = Name { last, first };
    println!("Hello, {}, {}", name.first, name.last);
}
```

Notice we access our struct members with dot notation, and there is no default new constructor or overloading in Rust. Though in practice, for functions where it makes sense many developers will offer and even require usage of a `new` function, and also make their struct private to require the usage of a new or other constructor and so new is still a good place to start looking. String::new() totally exists and would have made you an empty string.

* extra credit: Figure out how to make our name struct private and force usage of a new function to construct.


But hey this seems wordy, lets just print our whole struct in one formatter.


```rust
println!("Hello, {}", name);
```

Results in:
```
   Compiling training v0.1.0 (/Users/firstlast/Downloads/training)
error[E0277]: `Name` doesn't implement `std::fmt::Display`
  --> src/main.rs:30:27
   |
30 |     println!("Hello, {}", name);
   |                           ^^^^ `Name` cannot be formatted with the default formatter
   |
   = help: the trait `std::fmt::Display` is not implemented for `Name`
   = note: in format strings you may be able to use `{:?}` (or {:#?} for pretty-print) instead
   = note: required by `std::fmt::Display::fmt`

error: aborting due to previous error

For more information about this error, try `rustc --explain E0277`.
error: Could not compile `training`.

To learn more, run the command again with --verbose.
```

Thats actually really rather helpful error with a several ideas for fixing it. AND in this case if say you're offline and can't google for more information it even has another page or two of content if you run the `rustc --explain E0277` command it mentions! 

> The compiler in Rust is almost always, really, actually, trying to tell you whats wrong, AND how to fix it. And if you find a case where it is not the Rust community would likely want to know how to make that error and the resulting action you should take more clear. 

So we should either use the Debug trait by changing our `{}` to `{:?}` or implement the Display trait. Let's take a moment to introduce Traits and then implement the Display trait for our Name struct.


## let's make traits
In Rust we stress composition over inheritence using [traits](https://doc.rust-lang.org/book/ch10-02-traits.html). Traits, much like header files, seperate the defintion from the implementation. Before we solve our Display problem by consuming someone elses trait definition, lets make our own convoluted example.

We'll make a trait that has one function that println! SHOUTS our name. Again, we won't implement here, we just define
```rust
pub trait Shout {
    fn shout(self);
}
```

self represents the instance of the type that this will be called on, and in our case will give us access to the first and last fields via self.first and self.last. So now lets make an 'implementation' of Shout for Name.
```rust
impl Shout for Name {
    fn shout(self) {
        println!(
            "{} {}",
            self.first.to_uppercase(),
            self.last.to_uppercase()
        );
    }
}
```

Now anytime this trait is in namespace, which in our case it is because it defined in this same file, it is available on all instances of Names. Lets call it.
```rust
name.shout();
```
and well should see
```bash
FIRST LAST
```
> Teh seperation of definition from implementation is incredbly powerful. This way if we make our trait public anyone downstream can customize our function for their archictecture or edge case. This keeps Rust from amongst other things passing around huge config structs full of lifecycle callbacks and other configuration overrides.

OK. So we can forget about shout. Lets implement a trait we don't own, the Display trait shipped with std Rust for our custom Name type. As a reminder above, it said
```
   = help: the trait `std::fmt::Display` is not implemented for `Name`
```
Looking in the std documentation we find [Display](https://doc.rust-lang.org/std/fmt/trait.Display.html) which wants us to make a function which will be passed our instance called self, and another arg called f.

Some new syntax again:
* We finally have a function here in fmt that returns something. Thats indicated by the `->` syntax the fn definition which returns whatever type is on the right hand side. We'll cover the Result type later.
* Hopefully you're ahead of me and found the write! macro definition linked right on that page. It is just like println!  but with two big changes. Instead of printing the formatted string it sticks it in its first argument and it returns one of those Result things where println! didnt return anything.
* And wait. The write! macro function doesnt have a semicolon at the end this time! Thats indicates its return argument is implicitly being returned from this function. We could instead explicitly return and write `let blah = write!(f, "({}, {})", self.x, self.y); return blah;`

With that info you should have what you need to finish implementing Display for Name.

## args
Let's take our name from the command line with runtime args instead of hard coding it at compile time. Search the standard library for [args](https://doc.rust-lang.org/std/env/fn.args.html)

Heres an explicit use (import) finally and a for loop we can add to our main:
```rust
use std::env;

for argument in env::args() {
    println!("{}", argument);
}
```

And then note you can pass args around cargo to the binary were trying to run like:
```bash
$ cargo run -- one-arg 2 three anotherarg
    Finished dev [unoptimized + debuginfo] target(s) in 0.01s
     Running `target/debug/training one-arg 2 three anotherarg`
target/debug/training
one-arg
2
three
anotherarg
Hello, world!
```

Just like C the first argument is the name of the binary and the rest are your arguments. You know where we're headed. Dig out two arguments from your command line and stick them in your name struct.



calibrate audience
what questions do you have about rust

test

use after move borrow checking
scopes
raii 

load a file or image
options results stretch 
match peek at enums

for loop modify (invert?) image
iterators
with combinators

type state
socket server state
different key or some input
typestate mqtt networking state

manages sockets server
cargo workspaces

rayon 
networking, http? mqtt
