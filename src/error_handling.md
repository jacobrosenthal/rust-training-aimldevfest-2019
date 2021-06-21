# Option and Result

Let's take our arguments from the command line with runtime args instead of hard coding it at compile time. Search the standard library for [args](https://doc.rust-lang.org/std/env/fn.args.html) finds args come in as a iterator of a collection. We'll talk about iterators later, but for now we can for loop over them, or get the `nth()` value. (starting from an empty main for just a second)

```rust ,ignore,no_run
use std::env; // explicit use (import) finally

fn main() {
    println!("{:?}", env::args().nth(0));
    println!("{:?}", env::args().nth(1));
    println!("{:?}", env::args().nth(2));
}
```

```bash
Some("target/debug/blah222")
Some("valve.png")
Some("out.png")
```

Just like C command line args the 0th argument is the name of the binary and the rest are your arguments. Hrm now we have this Some type wrapping our strings...

The problem is the nth argument may or may not be there... This is the original sin of C style languages. They allowed null values in the language and billions of dollars have been lost in hacks and crashes as a result.

Rust doesn’t have exceptions, but rather the [Result type](https://doc.rust-lang.org/std/result/index.html) which can be used to propagate either the error or the result and looks like this:
![Result Type](./images/result.png)

And Rust doesn't have Null but rather the [Option type](https://doc.rust-lang.org/std/option/enum.Option.html) which can be used to propagate either the value (Some), or the lack of one (None)
![Option Type](./images/option.png)

Were going to skip Result here for now, as our `nth()` method returns an Option, but they’re very similar in how they’re handled as they’re both implemented as enums.

There *is* a minimal runtime in Rust, which means if were not careful we can and will blow up at runtime. This is called a panic and is handled in the panic handler, which on hosted platforms includes unwinding and backtraces. You can fire it on purpose with `panic!()` or if you `unwrap()` on a None or Err value.

```rust ,ignore,no_run
use std::env;

fn main() {
    panic!("ded");
    println!("{:?}", env::args().nth(0));
    println!("{:?}", env::args().nth(1));
    println!("{:?}", env::args().nth(2));
}
```

Notice the program gets no furter than the panic and nothing is printed.

```rust ,ignore,no_run
use std::env;

fn main() {
    println!("{:?}", env::args().nth(0).unwrap());
    println!("{:?}", env::args().nth(1).unwrap());
    println!("{:?}", env::args().nth(2).unwrap());
}
```

```bash
"target/debug/blah222"
"valve.png"
"out.png"
```

**EXERCISE: Run our program again, this time not passing any command line arguments.**

```text
thread 'main' panicked at 'called `Option::unwrap()` on a `None` value', src/libcore/option.rs:347:21
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace.
```

Explicitly panicing like we see above is a fine option when the error is unrecoverable anyway like in this case when theres nothing intelligent we can do except maybe try to print a decent error message. If the default error message from binary isn't very good well often well use `.expect()` in order to provide a better one for our users.

```rust ,ignore,no_run
use std::env;

fn main() {
    println!("{:?}", env::args().nth(1).expect("You must pass an input file name as the first argument"));
    println!("{:?}", env::args().nth(2).expect("You must pass an output file name as the second argument"));
}
```

**EXERCISE: Run our program again, this time not passing the first and or second argument.**

```bash
$ cargo run -- valve.png 
    Finished dev [unoptimized + debuginfo] target(s) in 0.01s
     Running `target/debug/blah222 valve.png`
"valve.png"
thread 'main' panicked at 'You must pass an output file name as the second option to this program', src/main.rs:14:14
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
$ 
```

There's definately command line argument parsing libraries that will do all this for us, but for now it is good to understand what is happening under the hood.

**EXERCISE: Swap our hardcoded arguments from the previous page for our new command line arguments.**

Next chapter we'll look at the Image library, and load up an image with our filenames.
