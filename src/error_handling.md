# error handling

Looking at the image examples and documentation we find image::open takes a file path (a String converts nicely to file path) and returns an ImageResult<DynamicImage>

This is the [Result type](https://doc.rust-lang.org/std/result/index.html) we hinted at at the end of the traits chapter. (were going to dodge the concept of generics here for a bit, but image result is wrapping) and introduces another big concept [error handing](https://doc.rust-lang.org/book/ch09-00-error-handling.html) seen here as `unwrap()`.

so before we can get at our image we have to deal with this wrapping ImageResult type. 


Grabbing the entire [image example from the bottom of the readme](https://github.com/image-rs/image#61-opening-and-saving-images_)
```rust,ignore
fn main() {
    // Use the open function to load an image from a Path.
    // ```open``` returns a `DynamicImage` on success.
    let img = image::open("tests/images/jpg/progressive/cat.jpg").unwrap();

    // The dimensions method returns the images width and height.
    println!("dimensions {:?}", img.dimensions());

    // The color method returns the image's `ColorType`.
    println!("{:?}", img.color());

    // Write the contents of this image to the Writer in PNG format.
    img.save("test.png").unwrap();
}
```

Lets naively run this code (sans cat pictures)
```text
thread 'main' panicked at 'called `Result::unwrap()` on an `Err` value: IoError(Os { code: 2, kind: NotFound, message: "No such file or directory" })', src/libcore/result.rs:999:5
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace.
```

## Option 1, panic

There is a minimal runtime in Rust, which means if were not careful we can and will blow up at runtime. This is called a panic and is handled in the panic handler, which on hosted platforms includes undwinding and backtraces.

Explicitly panicing may be an option when the error is unrecoverable anyway like in examples like this where theres nothing intelligent wecan do except print a decent error message, and frankly this one is pretty decent.. If space constraints of storing strings in our binary isnt a problem well often well use `.expect("Please enter an image file as the first option to this program")` to futher refine the message.

For our `image::open` and for example code, this is probably the correct solution. However as you can expect in libraries and production code panic and expect are red flags. Rust is trying to help you write programs that don't exit randomly when running afterall.


## Option 2, return it

Another option is to make it someone elses problem by simply handing the result back up the chain. This is what we did earlier in our Display trait where we simply returned the entire result type without even looking at it. We can even do that here even though this is a main function, as Rust will unwrap it behind the scenes and print the error. 

Theres also a helper macro for this case in order to exit early, the [? operator](https://doc.rust-lang.org/book/ch09-02-recoverable-errors-with-result.html#a-shortcut-for-propagating-errors-the--operator). This was previously the [try! macro](https://doc.rust-lang.org/std/macro.try.html) but that has been deprecated though you may still see it in code.

So to return a result from our function instead of unwrapping perhaps we would change
```rust, ingore, no_run
use image::ImageError; //<--import the error type

fn main() -> Result<(), image::ImageError> {  //<- Success returns the unit type or an image error
    // Use the open function to load an image from a Path.
    // ```open``` returns a `DynamicImage` on success.
    let img = image::open("tests/images/jpg/progressive/cat.jpg")?; //<-- swap .unwrap() for ? operator

    // The dimensions method returns the images width and height.
    println!("dimensions {:?}", img.dimensions());

    // The color method returns the image's `ColorType`.
    println!("{:?}", img.color());

    // Write the contents of this image to the Writer in PNG format.
    img.save("test.png").unwrap();

    Ok(()) //<-- Explicity return Ok
}
```

## option 3, handle it
Or you can deal with it yourself locally, remap it to a different erorr, use control flow to try again.etc


So we often will exaustively match them with match, which is similar to switch
* The enum [match pattern](https://doc.rust-lang.org/rust-by-example/flow_control/match.html) 
```rust,ignore,no_run
match result {
    Some(x) => println!("Result: {}", x),
    None    => println!("Cannot divide by 0"),
}
```

* if errors are used in control flow and you only care about the presence combinators like [is_ok()](https://doc.rust-lang.org/std/result/enum.Result.html#method.is_ok) or [is_err()](https://doc.rust-lang.org/std/result/enum.Result.html#method.is_err) are handy
```rust,ignore,no_run
if result1.is_ok() && result2.is_err() {

}
```

The Result is actually an enum type so we lets take a full digression through enums and matching
