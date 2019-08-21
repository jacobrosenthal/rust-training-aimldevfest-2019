
# enums and matching and options
tagged unions for the cs folks, but implemented so you cant hurt yourself http://patshaughnessy.net/2018/3/15/how-rust-implements-tagged-unions
todo have craig go on and on about enums

As we just saw, the Result enum is a Std type with variants of only either Ok(Generic Value) or Err(Error)

As you can see.. enums can not only constrain input to one of a set of limited values, they can also hold values, makes them algabreaic datatypes

Rust uses these features in luie of exceptions Rust uses algabreaic enums to allow you to pass back some data as part of your Result Erorr type
https://doc.rust-lang.org/stable/core/result/enum.Result.html


configuration also uses enums heavily to constrain arugments.


Looking at the (DynamicImage](https://docs.rs/image/0.22.1/image/enum.DynamicImage.html) type we got back from Open (also an enum btw) library we have a [resize function that takes a filter](https://docs.rs/image/0.22.1/image/enum.DynamicImage.html#method.resize) which is an enum of 
```rust,no_run
pub enum FilterType {
    Nearest,
    Triangle,
    CatmullRom,
    Gaussian,
    Lanczos3,
}
```

So bring the last few lessons together and locating a cat picture, we might like to resize our image before we save it out, add this line before the img.save
```rust,ignore,no_run
use image::{FilterType, ImageError};

struct Opt {
    input_path: String,
    output_path: String,
    scale_filter: FilterType,
}

fn main() -> Result<(), ImageError> {
    let options = Opt {
        input_path: String::from("cat.jpg"),
        output_path: String::from("test.png"),
        scale_filter: FilterType::Triangle,
    };

    let img = image::open(options.input_path)?;

    let img = img.resize(32, 32, options.scale_filter);

    img.save(options.output_path).unwrap();

    Ok(())
}
```
todo mention shadowing

we really need to stop hardcoding all this.

We can even write traits for enums! That would let us do something like turn a runtime argument string into a typed filter enum

```rust,ignore,no_run
FilterType::from_str().unwrap()
```

This will come in handy here.. 


# runtime arguments
Let's take our options from the command line with runtime args instead of hard coding it at compile time. Search the standard library for [args](https://doc.rust-lang.org/std/env/fn.args.html)

Heres an explicit use (import) finally and a for loop we can add to our main:
```rust,no_run
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



So lets fix our examples to not hardcode input_path, output_path, scale_filter

