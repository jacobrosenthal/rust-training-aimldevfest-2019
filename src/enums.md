# enums, matching, options

tagged unions for the cs folks, but implemented so you cant hurt yourself http://patshaughnessy.net/2018/3/15/how-rust-implements-tagged-unions
todo have craig go on and on about enums

As we just saw, the Result enum is a Std type with variants of only either Ok(Generic Value) or Err(Error)

As you can see.. enums can not only constrain input to one of a set of limited values, they can also hold values, makes them algebraic datatypes

Rust uses these features in lieu of exceptions Rust uses algebraic enums to allow you to pass back some data as part of your Result Error type
https://doc.rust-lang.org/stable/core/result/enum.Result.html


configuration also uses enums heavily to constrain arguments.


Looking at the (DynamicImage](https://docs.rs/image/0.22.1/image/enum.DynamicImage.html) (enum) type we got back from `image::Open()` method we have a [resize function that takes a filter](https://docs.rs/image/0.22.1/image/enum.DynamicImage.html#method.resize) which is an enum of 
```rust,no_run
pub enum FilterType {
    Nearest,
    Triangle,
    CatmullRom,
    Gaussian,
    Lanczos3,
}
```


We can even write traits for enums! That would let us do something like turn a runtime command line argument string into a typed filter enum. For now lets just use a `String` 

Implement this trait and add an object to our Opt struct. We'd probably want to go back to our match statement we briefly touched on... match with some kind of text like "nearest" to `FilterType::Nearest`
```rust,ignore,no_run
trait FilterString {
    fn from_str(input: &str) -> FilterType;
}
```
so that this compiles
```rust,ignore,no_run
    let options = Opt {
        input_path: String::from("cat.jpg"),
        output_path: String::from("test.png"),
        scale_filter: FilterType::from_str("tri"),
    };
```
Todo need to cover string borrows

But wait.. what if I cant find the string that gets entered?? We need a way to say that. We COULD use an error, like ENOTFOUND, But theres another enum that specifically designed for that. [Option](https://doc.rust-lang.org/std/option/index.html). It's variants are Some(Val) or None and is perfect when looking for something that might return something, or might not.

Fix up our trait with an option by returning something like `Some(FilterType::Nearest)` and add an exaustive catch all case with _ that returns None
```rust,ignore,no_run
trait FilterString {
    fn from_str(input: &str) -> Option<FilterType>;
}
```

so that this compiles
```rust,ignore,no_run
    let options = Opt {
        input_path: String::from("cat.jpg"),
        output_path: String::from("test.png"),
        scale_filter: FilterType::from_str("tri").expect("couldn't find filter"),
    };
```















Its left as an exercise to the user to find a cat picture. But after that bringing the last few lessons together, we might like to resize our image before we save it out, add this line before the `img.save()`
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
