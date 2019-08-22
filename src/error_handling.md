# error handling

Looking at the image examples and documentation we find image::open takes a file path (a String converts nicely to file path) and returns an ImageResult<DynamicImage>

Rust has no exceptions and it very much doesn’t want you to stick an error in the return value. So this is the [Result type](https://doc.rust-lang.org/std/result/index.html) we hinted at at the end of the traits chapter. (were going to dodge the concept of generics here for a bit, but image result is wrapping) which finally brings us to Rust's concept of [error handing](https://doc.rust-lang.org/book/ch09-00-error-handling.html) seen here as `unwrap()`.


Grabbing the entire [image example from the bottom of the readme](https://github.com/image-rs/image#61-opening-and-saving-images_)
```rust,ignore
fn main() {
    // Use the open function to load an image from a Path.
    // ```open``` returns a `DynamicImage` on success.
    let img = image::open(options.input_path).unwrap();

    // The color method returns the image's `ColorType`.
    println!("{:?}", img.color());

    // Write the contents of this image to the Writer in PNG format.
    img.save(options.output_path).unwrap();
}
```

Lets naively run this code (sans cat picture, guaranteeing an error)
```text
thread 'main' panicked at 'called `Result::unwrap()` on an `Err` value: IoError(Os { code: 2, kind: NotFound, message: "No such file or directory" })', src/libcore/result.rs:999:5
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace.
```

## Option 1, panic

There is a minimal runtime in Rust, which means if were not careful we can and will blow up at runtime. This is called a panic and is handled in the panic handler, which on hosted platforms includes unwinding and backtraces.

Explicitly panicing may be an option when the error is unrecoverable anyway like in examples like this where theres nothing intelligent we can do except maybe try to print a decent error message. If space constraints of storing strings in our binary isn’t a problem well often well use `.expect("Please enter an image file as the first option to this program")` to further refine the message.

For our `image::open` and for example code, this panic is a decent solution as the resulting error is fairly descriptive. However, as you can expect, in libraries and production code panic and expect are red flags. Rust is trying to help you write programs that don't exit randomly while running after all.


## Option 2, return it

Another option is to make it someone else’s problem by simply handing the result back up the chain. This is what we did earlier in our Display trait where we simply returned the entire result type without even looking at it. We can even do that here even though this is a main function, as Rust will unwrap it behind the scenes and print the error.

There's also a helper macro for this case in order to exit early, the [? operator](https://doc.rust-lang.org/book/ch09-02-recoverable-errors-with-result.html#a-shortcut-for-propagating-errors-the--operator). This was previously the [try! macro](https://doc.rust-lang.org/std/macro.try.html) but that has been deprecated though you may still see it in code.

If we wanted to implement this solution here, perhaps we would change
```rust, ignore, no_run
use image::ImageError; //<--import the error type

fn main() -> Result<(), image::ImageError> {  //<- Success returns the unit type or an image error
    // Use the open function to load an image from a Path.
    // ```open``` returns a `DynamicImage` on success.
    let img = image::open(options.input_path)?; //<-- swap .unwrap() for ? operator

    // The color method returns the image's `ColorType`.
    println!("{:?}", img.color());

    // Write the contents of this image to the Writer in PNG format.
    img.save(options.output_path)?; //<-- swap .unwrap() for ? operator

    Ok(()) //<-- Explicitly return Ok
}
```

## option 3, handle it
Or you can deal with it yourself locally.

So we often will exhaustively match them with [match pattern](https://doc.rust-lang.org/rust-by-example/flow_control/match.html) which is very similar to an exhaustive switch statement.

But you dont want to be deep in a match statement for your sucess case so thats not a great solution here.
```rust, ignore, no_run
    match image::open(options.input_path) {
        Ok(img) => {
            println!("{:?}", img.color());
        }
        Err(error) => println!("error!: {:?}", error),
    };
}
```


```rust, ignore, no_run
    match image::open(options.input_path) {
        Ok(img) => {
            println!("{:?}", img.color());
        }
        Err(error) => println!("error!: {:?}", error),
    };
}
```

```rust,ignore,no_run

let img_result = image::open(options.input_path); // no unwrap

if img_result.is_ok() {
            println!("{:?}", img.color());

}
```


Or if errors are used in control flow and you only care about the presence, combinators like [is_ok()](https://doc.rust-lang.org/std/result/enum.Result.html#method.is_ok) or [is_err()](https://doc.rust-lang.org/std/result/enum.Result.html#method.is_err) are handy:
```rust,ignore,no_run
if result1.is_ok() && result2.is_err() {

}
```

The Result is actually an enum type so we lets take a full digression through enums and matching in the next section.
