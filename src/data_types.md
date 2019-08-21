# Strings and println!

Weve got all the datatypes you would expect but you might want to glance through the Rust book [chapter on variables, functions, and control flow](https://doc.rust-lang.org/book/ch03-01-variables-and-mutability.html) just to update your mental models to Rust notation

We have signed and unsigned scalar types like u32 and i32 and we've got Strings. Variables are instantiated with let syntax, and notably are immutable by default.

The top of the [Rust standard library page](https://doc.rust-lang.org/std/) has a search box. Entering String there we find [std::string::String](https://doc.rust-lang.org/std/string/struct.String.html#method.from) with a bunch of example usage right there for us. You can also click the src button and be taken straight to the Rust implementation and you can even edit those examples and run them right in your browser to confirm your understanding.

> While you totally can thrash around on stack overflow, and we all do, there really is an authoratative source that you should check first.

From there we have our String constructor:
```rust,no_run
let input_path = String::from("cat.jpg");
```

First, note we dont need to import anything (we call it `use`) to use this type. A portion of the standard library is in our namespace automatically, which we call the [prelude](https://doc.rust-lang.org/std/prelude/index.html). Basically Rust puts `use std::prelude::v1::*;` at the top of your file and you get access to those members. By no means is everything in there, but a lot is, which is what kept you from explicitly writing `use std::string::String` at the top of your file in this case.

Also notice we didn't have to explicitly type our variable named first. What Rust *can* figure it out, *it will* and so its entirely idiomatic to omit type annotations. However if you or the compiler are having trouble or getting odd type errors, start annotating some of your types like to see if you can give the compiler a hand. Its also a great way to figure out what type you actually have in case you're not sure, let the compiler (or linter) tell you.

```rust,no_run
let output_path:String = String::from("test.png");
```

In Rust our printf replacement character is `{}`. Following the println! documentation down the rabbit hole will send us to the [formatters section](https://doc.rust-lang.org/std/fmt/index.html) page and we find all the formatters which you would expect like hex `{:x}`, binary `{:b}`, etc. We're going to focus on the 'empty' Display formatter `{}` for now which is a kind of a pretty printer in Rust. As long as whover wrote our type implemented the Display pretty printer trait this will work great (cue ominous music).
```rust,editable
fn main() {
    let input_path = String::from("cat.jpg");
    let output_path = String::from("test.png");
    println!("input_path:{} output_path:{}", input_path, output_path);
}
```
String pretty printing results in rather clean output in this case:
```text
input_path:cat.jpg output_path:test.png
```

Objects, we call them structs, should be very familiar. You can define a new struct in any scope you like and we can name and type their members.
```rust,editable
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
