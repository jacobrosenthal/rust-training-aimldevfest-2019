# Extra exercises

## arguments libraries
We could polish up this binary with some better command line argument parsing, error messages, version, etc, but if you were thinking someone else has to have done this type of work before, you'd be right. Theres a helper called [structopt](https://crates.io/crates/structopt) that uses [macros](https://doc.rust-lang.org/stable/book/ch19-06-macros.html) to annotate your existing struct. 
```rust,ignore,no_run
use std::path::PathBuf;
use structopt::StructOpt;

#[derive(StructOpt, Debug)]
#[structopt(name = "training")]
struct Opt {
    #[structopt(short = "i", long = "input", parse(from_os_str))]
    input_path: PathBuf,
    #[structopt(short = "o", long = "output", parse(from_os_str))]
    output_path: PathBuf,
}
```
Then not too much changes in our existing main
```rust,ignore,no_run
fn main() {
    let opt = Opt::from_args();
    println!("{:?}", opt);
}
```
Running `cargo run -- -i cat.jpg -o test.png` results in
```text
Opt { input_path: "cat.jpg", output_path: "test.png" }
```
and running `cargo run -- --help`
```text
training 0.1.0
First Last <FirstL@gmail.com>

USAGE:
    training --input <input_path> --output <output_path>

FLAGS:
    -h, --help       Prints help information
    -V, --version    Prints version information

OPTIONS:
    -i, --input <input_path>      
    -o, --output <output_path>    
```

## Cross compiling
We get cross compiling generally for free. The hard part is usually finding a linker. Check our recent meetup post on [cross compiling to a raspberry pi](https://rust.azdevs.org/2019-07-24/) But the general gist is, when you're tooling is set up its can be as easy as `cargo build --target armv7-unknown-linux-gnueabihf`

## ffi
Rust can consume C ABI or freeze its output to C ABI, so anything (Golang, Python, Node) could consume Rust code, and Rust could consume any C, C++, etc that you have already built. [rust-ffi-examples](https://github.com/alexcrichton/rust-ffi-examples) has examples for all your favorite languages.

For compiling C to Rust, you'll be using [rust-bindgen](https://github.com/rust-lang/rust-bindgen) which has the [bindgen user guide](https://rust-lang.github.io/rust-bindgen/introduction.html) for explaining its usage.

Generally speaking, we can get some Rust bindings generated 'for free' from the C headers. IE given the C header doggo.h bindgen can (often) produce a Rust module you can call from your existing code.

Its common to run bindgen as a bash command line you can maintain your upstream by live patch your header file. Then when confident you can hook into the [Rust build system](https://doc.rust-lang.org/cargo/reference/build-scripts.html) to script bindgen as well as [gcc](https://crates.io/crates/cc) or cc in a build.rs to create (and cache) artifacts as part of your build process.

Common libraries, often dynamically linked, exist in the crates.io repository, denoted with by -sys naming, with higher level cleaner apis are generally written separately on top of these packages.
* https://crates.io/crates/openssl-sys
* https://crates.io/crates/libz-sys
* https://crates.io/crates/curl-sys

You’ll want to get well acquainted with the [unsafe keyword](https://doc.rust-lang.org/book/ch19-01-unsafe-rust.html) heavily here as you cross the C to Rust border, as well as the [nomicon](https://doc.rust-lang.org/nomicon/ffi.html) to understand the darker arts.

You can also go the other way, call your Rust code from C. A common helper library is [cbindgen](https://github.com/eqrion/cbindgen)

## type states

## generics

## networking

