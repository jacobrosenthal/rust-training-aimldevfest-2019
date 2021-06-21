# arguments libraries

We could polish up this binary with some better command line argument parsing, error messages, version, etc, but if you were thinking someone else has to have done this type of work before, you'd be right. Theres a helper called [structopt](https://crates.io/crates/structopt) that uses [macros](https://doc.rust-lang.org/stable/book/ch19-06-macros.html) to annotate your existing struct.

```rust,ignore,no_run
use std::path::PathBuf;
use structopt::StructOpt;

#[derive(StructOpt, Debug)]
#[structopt(name = "training")]
struct Arguments {
    #[structopt(short = "i", long = "input", parse(from_os_str))]
    input_path: PathBuf,
    #[structopt(short = "o", long = "output", parse(from_os_str))]
    output_path: PathBuf,
}
```

Then not too much changes in our existing main.

```rust,ignore,no_run
fn main() {
    let arguments = Arguments::from_args();
    println!("{:?}", arguments);
}
```

Running `cargo run -- -i valve.png -o valve_sobel.png` results in

```text
Arguments { input_path: "valve.png", output_path: "valve_sobel.png" }
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
