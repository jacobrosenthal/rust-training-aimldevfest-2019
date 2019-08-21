# error handling, the result type
Lets add our input and output to the example from the image crate and learn some more new concepts
```rust,ignore
//todo
use image::GenericImageView;

let img = image::open(options.input_path).unwrap();

// The dimensions method returns the images width and height.
println!("dimensions {:?}", img.dimensions());

// The color method returns the image's `ColorType`.
println!("{:?}", img.color());

// Write the contents of this image to the Writer in PNG format.
img.save(options.output_path).unwrap();
```
Theres one new big concept here we skipped earlier, [error handing](https://doc.rust-lang.org/book/ch09-00-error-handling.html) seen here as `unwrap()`.

There is a minimal runtime in Rust, which means if were not careful we can and will blow up at runtime. This is called a panic and is handled in the panic handler, which on hosted platforms includes undwinding and backtraces.

Lets naively run our example (still missing a cat picture)
```text
thread 'main' panicked at 'called `Result::unwrap()` on an `Err` value: IoError(Os { code: 2, kind: NotFound, message: "No such file or directory" })', src/libcore/result.rs:999:5
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace.
```

## Option 1, panic

panic may be an option when the error is unrecoverable anyway like in examples like this where its pretty descriptive, but in libraries and production code this is a huge red flag. Rust is trying to help you write programs that don't exit randomly while running afterall. For our `image::open` this is probably a fine solution in this case.


## Option 2, return it

Methods that can error utilize the [Result type](https://doc.rust-lang.org/std/result/index.html) to pass return either the error or the result back to the calling site and further Rust's default is to force you to explicitly handle the handle the error case.

Another option is to make it someone elses problem by simply handing the result back up the chain. This is what we did earlier in our Display trait where we simply returned the entire result type without even looking at it. We can do that here even though this is a main function, as Rust will unwrap it behind the scenes, so lets do that.

return a result from our function instead of unwrapping perhaps we would change
```rust, ingore, no_run
use image::GenericImageView, ImageError; //<--import the error type

fn main() -> Result{ //<- add the function return
    let img = image::open("tests/images/jpg/progressive/cat.jpg")?; //<-- swap .unwrap() for ?
```
Theres also a helper macro for this case in order to exit early, the [? operator](https://doc.rust-lang.org/book/ch09-02-recoverable-errors-with-result.html#a-shortcut-for-propagating-errors-the--operator). This was previously the [try! macro](https://doc.rust-lang.org/std/macro.try.html) but that has been deprecated though you may still see it in code.

We also get exit from main for free on hosted operating systems, so we can simply have main return the result to be printed on the console. So we could change our example like so
```rust,ignore
use image::GenericImageView;
//todo
fn main() -> Result { //<- added the return
    // Use the open function to load an image from a Path.
    // ```open``` returns a `DynamicImage` on success.
    let img = image::open("tests/images/jpg/progressive/cat.jpg")?; //<-- replaced unwrap

    // The dimensions method returns the images width and height.
    println!("dimensions {:?}", img.dimensions());

    // The color method returns the image's `ColorType`.
    println!("{:?}", img.color());

    // Write the contents of this image to the Writer in PNG format.
    img.save("test.png").unwrap();
}
```

We used to use  Nowadays we cant use the try operator though, it has been replaced with the 

## option 3, handle it
Or you can deal with it yourself
* The old school [match pattern](https://doc.rust-lang.org/rust-by-example/flow_control/match.html) 
```rust,ignore,no_run
match result {
    Some(x) => println!("Result: {}", x),
    None    => println!("Cannot divide by 0"),
}
```

* [if let destructuring](https://doc.rust-lang.org/rust-by-example/flow_control/if_let.html) can be nicer than match sometimes
```rust,ignore,no_run
if let Some(i) = number {
    println!("Matched {:?}!", i);
}
```

* if you dont care about the value or error just the end (little case r) result try [is_ok()](https://doc.rust-lang.org/std/result/enum.Result.html#method.is_ok) or [is_err()](https://doc.rust-lang.org/std/result/enum.Result.html#method.is_err)
```rust,ignore,no_run
if(x.is_ok() && y.is_err()){
}
```

* you can also use [Combinators](https://learning-rust.github.io/docs/e6.combinators.html) to string a bunch of results or options together. Theres a TON of these
```rust,ignore,no_run
s1.or(s2)
s1.or_else(|| Some("some2"))
s1.and_then(|| Some("some2"))
```

