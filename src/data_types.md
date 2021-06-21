# Data types

On this section well explore data types like String, structs, function syntax, and make a slightly convoluted example to hold the input and output file names that we'll take via command line for our images later.

```rust ,ignore,no_run
struct Arguments {
    input_path: String,
    output_path: String,
}

fn arguments() -> Arguments {
    Arguments {
        input_path: String::from("valve.png"),
        output_path: String::from("valve_sobel.png"),
    }
}

fn main() {
    let arguments = arguments();
    println!("{} {}", arguments.input_path, arguments.output_path);
}
```

We’ve got all the datatypes you would expect but you might want to glance through the Rust book [chapter on variables, functions, and control flow](https://doc.rust-lang.org/book/ch03-01-variables-and-mutability.html) just to update your mental models to Rust notation

We have signed and unsigned scalar types like u32 and i32 and we've got Strings. Variables are instantiated with let syntax, and notably are immutable by default.

The top of the [Rust standard library page](https://doc.rust-lang.org/std/) has a search box. Entering String there we find [std::string::String](https://doc.rust-lang.org/std/string/struct.String.html#method.from) with a bunch of example usage right there for us. You can edit those examples and run them right in your browser to confirm your understanding and even click the [src] link in the upper right corner and be taken straight to the Rust implementation.

> While you totally can thrash around on stack overflow, and we all do, there really is an authoritative source that you should check first.

From that example we have our `String::from` constructor. Lets assume we saved an image file as "valve.png" that we'll eventually open, passing that from command line. For now lets just hardcode our filename.

```rust ,editable
fn main() {
    let input_path = String::from("valve.png");
    println!("Hello, world!");
}
```

First, note we don’t need to import anything (we call it `use`) to use this `String` type. A portion of the standard library is in our namespace automatically, which we call the [prelude](https://doc.rust-lang.org/std/prelude/index.html). Basically Rust puts `use std::prelude::v1::*;` at the top of your file and you get access to those members. By no means is everything in there, but a lot is, which is what kept you from explicitly writing `use std::string::String` at the top of your file in this case. The `println!` macro came from there as well.

<details><summary>Digression on cargo-expand</summary>
<p>

Out of the scope for this workshop, but if you wanted to install a cargo tool called [cargo-expand](https://github.com/dtolnay/cargo-expand) you could see the expanded result of your code with all macros and preludes included but before it has been optimized to machine code.

```bash
$ cargo install cargo-expand
..
$ cargo expand
#![feature(prelude_import)]
#[prelude_import]
use std::prelude::rust_2018::*;
#[macro_use]
extern crate std;
fn main() {
    let input_path = String::from("valve.png");
    {
        ::std::io::_print(::core::fmt::Arguments::new_v1(
            &["Hello, world!\n"],
            &match () {
                () => [],
            },
        ));
    };
}
$
```

You don't have to understand all that, but just to show if Rust ever feels magic, there are tools to look under the hood to see whats going on.

</p>
</details>

Also notice we didn't have to explicitly type our input_path variable even though Rust is a typed language. What Rust *can* figure it out, *it will* and so its entirely idiomatic to omit type annotations. However if you or the compiler are having trouble or getting odd type errors, start annotating some of your types like to see if you can give the compiler a hand. Its also a great way to figure out what type you actually have in case you're not sure, let the compiler (or linter) tell you.

Lets assume that we want to store our output name as "valve_sobel.png". The compiler doesn't need it in this instance but we can We can add a type annotation after the variable name with a colon and type like `:String`.

```rust ,editable
fn main() {
    let input_path = String::from("valve.png"); // <- no type annotation
    let output_path: String = String::from("valve_sobel.png"); // <- type annotation
    println!("Hello, world!");
}
```

So now how to print those variables to console instead of that useless "hello world".  In Rust our formatting character is `{}`. Following the `println!()` documentation down the rabbit hole will send us to the [formatters section](https://doc.rust-lang.org/std/fmt/index.html) page and we find all the formatters which you would expect like hex `{:x}`, binary `{:b}` etc. We're going to focus on the Debug formatter `{:?}` for now which is almost always implemented for types in Rust though the output may not be pretty.

```rust ,editable
fn main() {
    let input_path = String::from("valve.png");
    let output_path = String::from("valve_sobel.png");
    println!("{:?} {:?}", input_path, output_path);
}
```

```text
"valve.png" "valve_sobel.png"
```

Objects, we call them structs, should be very familiar. You can define a variable and construct a struct in any scope you like and we can name and type their members. This is somewhat a convoluted example, but lets make a struct called `Arguments` to hold our stubbed command line arguments.

```rust ,editable
// definition of our struct
struct Arguments {
    // name   : type
    input_path: String,
    output_path: String,
}

fn main() {

    // manully contstruct an instance of our struct and name it arguments
    let arguments = Arguments {
        // well keep using our hardcoded names for now until we learn how to get arguments from the command line
        input_path: String::from("valve.png"),
        output_path: String::from("valve_sobel.png"),
    };

    println!("{:?} {:?}", arguments.input_path, arguments.output_path);
}
```

Notice we access our struct members with dot notation, and there is no default new constructor or overloading in Rust. Though in practice, for functions where it makes sense many developers will offer and occasionally make their struct private to require the usage of a new or other named constructor. For instance earlier, `String::new()` totally exists and would have made you an empty string, however there is no way to manually construct a string like `let input_path = String { something: "valve.png" };`

Lets start modularizing our main by putting our argument creation into a function. Function syntax is just like we see in the main function.

```rust ,editable
// name        -> return type
fn arguments() -> Arguments {

    let arguments = Arguments {
        input_path: String::from("valve.png"),
        output_path: String::from("valve_sobel.png"),
    };
    return arguments; // <- explicit return statement
}
```

We use semicolons to end expressions. We prefer to leave off semicolons in order to implicitly return the expression saving in a temporary variable like so.

**EXERCISE: Put everything together and call our arguments() function.**

```rust ,editable
struct Arguments {
    input_path: String,
    output_path: String,
}

fn arguments() -> Arguments {
    Arguments {
        input_path: String::from("valve.png"),
        output_path: String::from("valve_sobel.png"),
    }
}

fn main() {
    let arguments = arguments();
    println!("{:?} {:?}", arguments.input_path, arguments.output_path);
}
```

In our next chapter, we'll get our actual arguments from the command line, and learn about error handling in Rust.
