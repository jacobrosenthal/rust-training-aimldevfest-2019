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
We can generally cross compile 'for free' if rust already has the target were interested in. [Rust Platform Support](https://forge.rust-lang.org/platform-support.html) is a great tool to see the status of various targets.

To get one of those supported targets on our machine all we have to do is `rustup target add` and we can often just use system LLVM to link most all bare metal no_std (freestanding or unhosted) targets like Cortex devices. And if not cross platform toolchains are generally very available. See the [Rust Embedded book](https://rust-embedded.github.io/book/intro/install/macos.html) for more on these targets.

As an aside, if you want to check out no_std raspberry pi stuff, I recommend [Phillip Oppermans blog series](http://os.phil-opp.com) which is basically a CS Degree in building an operating system from scratch on the raspberry pi.

For hosted linux on the raspberry pi though `rustup target add armv7-unknown-linux-gnueabihf` gets us started. Now it should be as easy as `cargo build --target armv7-unknown-linux-gnueabihf` but if we try that well see
```bash
   Compiling pi-example v0.1.0 (/Users/jacobrosenthal/Downloads/pi-example)
error: linker `arm-linux-gnueabihf-gcc` not found
  |
  = note: No such file or directory (os error 2)
error: aborting due to previous error
error: Could not compile `pi-example`.
To learn more, run the command again with --verbose.
```

So we clearly need a arm-linux-gnueabihf-gcc. If were in linux its probably actually [not that hard to get the raspberry pi toolchain linker](https://hackernoon.com/compiling-rust-for-the-raspberry-pi-49fdcd7df658). Something like `sudo apt-get install gcc-4.7-multilib-arm-linux-gnueabihf` would probably work

Then create a .cargo/config file and add the following to specify the linker, and the default target so you don’t have to specify --target every time.
```text
[build]
target = "armv7-unknown-linux-gnueabihf"

[target.armv7-unknown-linux-gnueabihf]
linker = "arm-linux-gnueabihf-gcc-4.7"
```

and now `cargo build` should work for you! 

But on Mac or Windows that toolchain is for some reason just not commonly hosted anywhere. The best Ive found is this million page medium article [Setup GCC 8.1 Cross Compiler Toolchain for Raspberry Pi 3 on macOS High Sierra](https://medium.com/coinmonks/setup-gcc-8-1-cross-compiler-toolchain-for-raspberry-pi-3-on-macos-high-sierra-cb3fc8b6443e)

But you know whats better than doing all that yourself and polluting your machine? Having someone else do work for you and packaging it in a reusable cross platform way. Maybe with something like .. a docker container.. Enter [Cross](https://github.com/rust-embedded/cross)

Cross hasn’t had an update in a while, so I recommend installing from git head with: `cargo install --force --git https://github.com/rust-embedded/cross cross`

And assuming your target is supported you can simply swap your cargo command for a cross command like
```bash
cross build --target=armv7-unknown-linux-gnueabihf
```

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

