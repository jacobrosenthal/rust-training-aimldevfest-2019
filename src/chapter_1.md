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

## Strings and println!

Weve got all the datatypes you would expect but you might want to glance through the Rust book [chapter on variables, functions, and control flow](https://doc.rust-lang.org/book/ch03-01-variables-and-mutability.html) just to update your mental models to Rust notation

We have signed and unsigned scalar types like u32 and i32 and we've got Strings. Variables are instantiated with let syntax, and notably are immutable by default.

The top of the [Rust standard library page](https://doc.rust-lang.org/std/) has a search box. Entering String there we find [std::string::String](https://doc.rust-lang.org/std/string/struct.String.html#method.from) with a bunch of example usage right there for us. You can also click the src button and be taken straight to the Rust implementation and you can even edit those examples and run them right in your browser to confirm your understanding.

> While you totally can thrash around on stack overflow, and we all do, there really is an authoratative source that you should check first.

From there we have our String constructor:
```rust
let input_path = String::from("cat.jpg");
```

First, note we dont need to import anything (we call it `use`) to use this type. A portion of the standard library is in our namespace automatically, which we call the [prelude](https://doc.rust-lang.org/std/prelude/index.html). Basically Rust puts use `std::prelude::v1::*;` at the top of your file and you get access to those members. By no means is everything in there, but a lot is, which is what kept you from explicitly writing `use std::string::String` at the top of your file in this case.

Also notice we didn't have to explicitly type our variable named first. What Rust *can* figure it out, *it will* and so its entirely idiomatic to omit type annotations. However if you or the compiler are having trouble or getting odd type errors, start annotating some of your types like to see if you can give the compiler a hand. Its also a great way to figure out what type you actually have in case you're not sure, let the compiler (or linter) tell you.

```rust
let output_path:String = String::from("test.png");
```

In Rust our printf replacement character is `{}`. Following the println! documentation down the rabbit hole will send us to the [formatters section](https://doc.rust-lang.org/std/fmt/index.html) page and we find all the formatters which you would expect like hex `{:x}`, binary `{:b}`, etc. We're going to focus on the 'empty' Display formatter `{}` for now which is a kind of a pretty printer in Rust. As long as whover wrote our type implemented the Display pretty printer trait this will work great (cue ominous music).
```rust
fn main() {
    let input_path = String::from("cat.jpg");
    let output_path = String::from("test.png");
    println!("input_path:{} output_path:{}", input_path, output_path);
}
```
String pretty printing results in rather clean output in this case:
```bash
input_path:cat.jpg output_path:test.png
```

Objects, we call them structs, should be very familiar. You can define a new struct in any scope you like and we can name and type their members.
```rust
struct Opt {
    input_path: String,
    output_path: String,
}

fn main() {
    let input_path = String::from("cat.jpg");
    let output_path = String::from("test.png");

    let options = Opt {
        input_path,
        output_path,
    };
    println!(
        "input_path:{} output_path:{}",
        options.input_path, options.output_path
    );
}
```

Notice we access our struct members with dot notation, and there is no default new constructor or overloading in Rust. Though in practice, for functions where it makes sense many developers will offer and even require usage of a `new` function, and also make their struct private to require the usage of a new or other constructor and so new is still a good place to start looking. String::new() totally exists and would have made you an empty string.

But hey this seems wordy, lets just print our whole struct in one formatter.
```rust
println!("{}", options);
```

Running this results in:
```
error[E0277]: `Opt` doesn't implement `std::fmt::Display`
  --> src/main.rs:14:20
   |
14 |     println!("{}", options);
   |                    ^^^^^^^ `Opt` cannot be formatted with the default formatter
   |
   = help: the trait `std::fmt::Display` is not implemented for `Opt`
   = note: in format strings you may be able to use `{:?}` (or {:#?} for pretty-print) instead
   = note: required by `std::fmt::Display::fmt`

error: aborting due to previous error

For more information about this error, try `rustc --explain E0277`.
error: Could not compile `training`.

To learn more, run the command again with --verbose.
```

Thats actually really rather helpful error with a several ideas for fixing it. AND in this case if say you're offline and can't google for more information it even has another page or two of content if you run the `rustc --explain E0277` command it mentions! 

> The compiler in Rust is almost always, really, actually, trying to tell you whats wrong, AND how to fix it. And if you find a case where it is not the Rust community would likely want to know how to make that error and the resulting action you should take more clear. 

So we should either use the Debug trait by changing our `{}` to `{:?}` or implement the Display trait. Let's take a moment to introduce Traits and then implement the Display trait for our Opt struct.

## traits
In Rust we stress composition over inheritence using [traits](https://doc.rust-lang.org/book/ch10-02-traits.html). Traits, much like header files, seperate the defintion from the implementation. Before we solve our Display problem by consuming someone elses trait definition, lets make a convoluted example to illustrate the syntax

We'll make a trait that has one function that println! SHOUTS our options. Again, we won't implement here, we just define
```rust
pub trait Shout {
    fn shout(self);
}
```

self represents the instance of the type that this will be called on, and in our case will give us access to the first and last fields via self.input_path and self.output_path. So now lets make an 'implementation' of Shout for Opt.
```rust
impl Shout for Opt {
    fn shout(self) {
        println!(
            "{} {}",
            self.input_path.to_uppercase(),
            self.output_path.to_uppercase()
        );
    }
}
```

Now anytime this trait is in namespace, which in our case it is because it defined in this same file, it is available on all instances of Opt. Lets call it.
```rust
options.shout();
```
and well should see
```bash
CAT.JPG TEST.PNG
```
> The seperation of definition from implementation is incredbly powerful. This way if we make our trait public anyone downstream can customize our function for their archictecture or edge case. This keeps Rust from amongst other things passing around huge config structs full of lifecycle callbacks and other configuration overrides.

OK. So we can forget about shout. Lets implement a trait we don't own, the Display trait shipped with std Rust for our custom Opt type. As a reminder above, it said
```
   = help: the trait `std::fmt::Display` is not implemented for `Opt`
```
Looking in the std documentation we find [Display](https://doc.rust-lang.org/std/fmt/trait.Display.html) which wants us to make a function which will be passed our instance called self, and another arg called f.

Some new syntax again:
* We finally have a function here in fmt that returns something. Thats indicated by the `->` syntax the fn definition which returns whatever type is on the right hand side. We'll cover the Result type later.
* Hopefully you're ahead of me and found the write! macro definition linked right on that page. It is just like println!  but with two big changes. Instead of printing the formatted string it sticks it in its first argument and it returns one of those Result things where println! didnt return anything.
* And wait. The write! macro function doesnt have a semicolon at the end this time! Thats indicates its return argument is implicitly being returned from this function. We could instead explicitly return and write `let blah = write!(f, "({}, {})", self.x, self.y); return blah;`

With that info you should have what you need to finish implementing Display for Opt.

## runtime arguments
Let's take our options from the command line with runtime args instead of hard coding it at compile time. Search the standard library for [args](https://doc.rust-lang.org/std/env/fn.args.html)

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

Just like C the first argument is the name of the binary and the rest are your arguments. You know where we're headed. Dig out first arguments from your command line and stick them in your options struct.

## external depdencies, crates.io
Lets have our code load in an image from the filesystem. Searching in the standard library for images doesn't find anything, we could take a File to binary, but lets go to the community ecosystem, [crates.io](https://www.crates.io). Searching there for images finds a crate image with ~1mil downloads which seems to be pretty popular. [image](https://crates.io/crates/image) says it wants us to add it to our Cargo.toml dependencies section so lets do that.

```toml
[dependencies]
image = "0.22.1"
```

The Cargo toml manifest version field is described here https://doc.rust-lang.org/cargo/reference/manifest.html#the-version-field where we learn Cargo uses [semantic versioning](https://semver.org) which allows us to version and lock dependencies at the level of risk were comfortable with. From the spec:
```
Given a version number MAJOR.MINOR.PATCH, increment the:

MAJOR version when you make incompatible API changes,
MINOR version when you add functionality in a backwards compatible manner, and
PATCH version when you make backwards compatible bug fixes.
```

The [Cargo chapter on dependencies](https://doc.rust-lang.org/cargo/reference/specifying-dependencies.html) explains more how to do this locking. The three digit version we used above is the same as a [caret requirement](https://doc.rust-lang.org/cargo/reference/specifying-dependencies.html#caret-requirements) as if we had type `image = "^0.22.1"`. With this requirement Cargo is allowed to use any version it can satisfy between the range `>=0.22.1 <0.3.0` Semver works different below and above 1.0 with the idea that theres more breaking churn below 1.0. So for a fictional `image = "^1.2.3"` Cargo would be allowed to find patches `>=1.2.3 <2.0.0`. Refer to the spec and the book for many more clarifying examples.

The most restrictive version would be `image = "= 0.22.1` which would not allow cargo any update capability. This can be handy for to make production code reproducable. Further along that line the resolved version state of all your dependencies (recursively) is captured in the Cargo.lock file and for binaries like ours can and should be checked into the repository. This way even if you're not locking the version explicitely you're still tracking and reviewing the upstreaming of all version changes. Finally, and outside of scope here you may also use [cargo vendor](https://doc.rust-lang.org/stable/cargo/commands/cargo-vendor.html) to download all your deps locally and check them into your repository and or you may host your own [alternate registry](https://doc.rust-lang.org/cargo/reference/registries.html) in which you only publish vetted versions.


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
