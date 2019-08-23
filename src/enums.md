# enums, matching, options
So we’ve seen enums are good at constraining a type between a limited set of values and they can also hold values, like an error or a type, which makes them algebraic datatypes. Rust Enums are liked [tagged unions for the C](http://patshaughnessy.net/2018/3/15/how-rust-implements-tagged-unions) folks but implemented in such a way that you cant hurt yourself.

TODO have craig go on and on about enums

Looking through the image documentation, we can [open an image](https://docs.rs/image/0.22.1/image/fn.open.html), get back a ImageResult containing a [DynamicImage type](https://docs.rs/image/0.22.1/image/enum.DynamicImage.html).
```rust,ignore,no_run
//just unwrap our Result as written we have to have a value or we would have already blown up
let img = image::open(options.input_path).unwrap();
```

Then we can use any of the many handy methods including a [resize method](https://docs.rs/image/0.22.1/image/enum.DynamicImage.html#method.resize) Authors tend to reach for enums often in constraining input to functions. Here FilterType Enum could be one of the following sampling filter:
```rust,no_run
pub enum FilterType {
    Nearest,
    Triangle,
    CatmullRom,
    Gaussian,
    Lanczos3,
}
```

So something like
```rust,ignore,no_run
//using the same variable name, called shadowing, is often even encouraged, as it means less messy temporary variables.
let img = img.resize(32, 32, FilterType::Nearest);
```
before finally saving out like:
```rust,ignore,no_run
img.save(options.output_path).unwrap();
```

Finding a cat picture and assembling the pieces is left as a exercise for reader.


So obviously we'd like to take resize from the command line, which means wed like a match statement to go from a command line argument String to a FilterType Enum, and we need to update our Opt struct to hold it. Wed like to resize based on command line input constrained to one of these types.
Naively we could implement the following:
```rust,ignore,no_run
use std::env;

struct Opt {
    input_path: String,
    output_path: String,
    scale_filter: FilterType,
}

fn options() -> Option<Opt> {
    let filter_string = env::args().nth(3)?;

    //we actually match on a as_ref borrow of the String
    let filter = match filter_string.as_ref() {
        "nearest" => FilterType::Nearest,
        "triangle" => FilterType::Triangle,
        "catmullrom" => FilterType::CatmullRom,
        "gaussian" => FilterType::Gaussian,
        "lanczos3" => FilterType::Lanczos3,
        _ => panic!("uhh I don’t know that filter"),
    };

    Some(Opt {
        input_path: env::args().nth(1)?,
        output_path: env::args().nth(2)?,
        scale_filter: filter,
    })
}

fn main() {
    let options = options().unwrap();
    println!("{}", options);
}
```

Which is totally workable but we can do one better, we can even write traits for enums which would be a clever solution to this problem. Lets abstract all that matching code into a trait. 

Heres a trait definition:
```rust,ignore,no_run
trait FilterString {
    fn from_str(input: String) -> Option<FilterType>;
}
```
and the usage
```rust,ignore,no_run
fn options() -> Option<Opt> {
    let filter_string = env::args().nth(3)?;

    Some(Opt {
        input_path: env::args().nth(1)?,
        output_path: env::args().nth(2)?,
        scale_filter: FilterType::from_str(filter_string)?,
    })
}
```
Now finish out the the FilterString impl to make all this work

# enum playground

You've started to aqaint yourself with enums in the error handling playground, but theres so much more it is worth spending some more time in the enum playground here to get your mind around how powerful the [pattern syntax](https://doc.rust-lang.org/book/ch18-03-pattern-syntax.html) is.
TODO change this stolen example from https://doc.rust-lang.org/book/ch06-02-match.html
```rust
enum UsState {
    Alabama,
    Alaska,
}

//enums can contain any other type
enum Coin {
    Penny,
    Nickel(u32),
    Dime(String),
    Quarter(UsState),
}

fn value_in_cents(coin: Coin) -> u8 {
    //you can match on any combination of your enum
    match coin {
        Coin::Penny => 1,
        Coin::Nickel(date) if date < 1920 => 50,
        Coin::Nickel(date) if date >= 1920 => 5,
        Coin::Dime(ref text) if text == "scratched" => 5,
        Coin::Dime(text) => {
            println!("{}", text);
            10
        }
        Coin::Quarter(UsState::Alaska) => {
            println!("State quarter from Alaska");
            25
        }
        Coin::Quarter(_state) => {
            println!("State quarter from elsewhere");
            25
        }
        //matches are exaustive, so if you don't cover all your use cases you need a catch all
        _ => 5,
    }
}

fn main() {
    println!("{}", value_in_cents(Coin::Quarter(UsState::Alabama)));
    println!("{}", value_in_cents(Coin::Dime(String::from("scratched"))));
    println!("{}", value_in_cents(Coin::Dime(String::from("A+"))));
    println!("{}", value_in_cents(Coin::Nickel(1921)));
    println!("{}", value_in_cents(Coin::Nickel(2000)));

    match 94 as u32 {
        1 | 2 => println!("one or two"),
        3...4 => println!("three or four"),
        11 => println!("11"),
        12..=44 => println!("12 to 44 inclusive"),
        _ => println!("The rest"),
    }
}
```