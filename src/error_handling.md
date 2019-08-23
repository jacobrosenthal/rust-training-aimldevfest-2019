# options and results


Let's take our config options from the command line with runtime args instead of hard coding it at compile time. Search the standard library for [args](https://doc.rust-lang.org/std/env/fn.args.html) finds args come in as a iterator of a collection. We'll talk about iterators later, but for now we can for loop over them, or get the `nth()` value. Just like C command line args the 0th argument is the name of the binary and the rest are your arguments. 
```rust,ignore, no_run
use std::env; // explicit use (import) finally

fn options() -> Opt {

    Opt {
        input_path: env::args().nth(1).unwrap(),
        output_path: env::args().nth(2).unwrap(),
    }
}
``` 

And then note you can pass args around cargo to the binary were trying to run like:
```bash
$ cargo run -- cat.jpg test.png
    Finished dev [unoptimized + debuginfo] target(s) in 0.01s
     Running `target/debug/training cat.jpg test.png `
target/debug/training
cat.jpg
test.png
```

So what is this unwrap. The problem is the nth argument may or may not be there.. 


We have two related types commingled in [error handling](https://doc.rust-lang.org/book/ch09-00-error-handling.html) in Rust. 


Rust doesn’t have exceptions, but rather the [Result type](https://doc.rust-lang.org/std/result/index.html) which can be used to propagate either the error or the result and looks like this:
```
pub enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

And Rust doesn't have Null but rather the [Option type](https://doc.rust-lang.org/std/option/enum.Option.html) which can be used to propagate either the value (Some), or the lack of one (None)
```
pub enum Option<T> {
    None,
    Some(T),
}
```

Were going to skip Result here, as our `nth()` method returns an Option, but they’re very similar in how they’re handled as they’re both implemented as enums. We basically have three possibilities for dealing with both:


## Option 1, panic! unwrap and expect

There *is* a minimal runtime in Rust, which means if were not careful we can and will blow up at runtime. This is called a panic and is handled in the panic handler, which on hosted platforms includes unwinding and backtraces. Run our program again, this time not passing arguments
```text
thread 'main' panicked at 'called `Option::unwrap()` on a `None` value', src/libcore/option.rs:347:21
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace.
```
Explicitly panicing like we see above may very well be an option when the error is unrecoverable anyway like in examples like this where theres nothing intelligent we can do except maybe try to print a decent error message. If the error message needs help well often well use `.expect("Please enter an image file as the first option to this program")` instead of `unwrap()` in order to further refine the message.


## Option 2, return it

Another option is to make it someone else’s problem by simply handing the Option or Result back up the chain. 

Theres even an exit early helper for this, the [? operator](https://doc.rust-lang.org/book/ch09-02-recoverable-errors-with-result.html#a-shortcut-for-propagating-errors-the--operator). This was previously the [try! macro](https://doc.rust-lang.org/std/macro.try.html) but that has been deprecated though you may still see it in code.

Our `nth()` is an Option of Some or None so lets just hand an optional back up the chain to our main function. At least this way we can decide what to do with it there.
```rust, ignore, no_run
fn options() -> Option<Opt> { //<-return Option wrapping our Opt struct

    Some(Opt {
        input_path: env::args().nth(1).expect("first argument input file"),
        output_path: env::args().nth(2).expect("second argument output file"),
    })
}
```

But now we've made it main's problem to deal with. Its silly but in this case, lets just unwrap there anyway. Were back to just blowing up, but at least our deep nested library or function call isn't blowing up, our top level binary is.
```rust, ignore, no_run
  let options = options().unwrap();
```


## option 3, handle it
Someone has to do some control flow on this error somewhere.. Well thats actually not true, we can even return these from the main function where Rust will unwrap them behind the scenes and print the result. 

But generally if you can do control flow on your errors you should. The way we often do that is through matching or combinators.

So we often will exhaustively match them with [match pattern](https://doc.rust-lang.org/rust-by-example/flow_control/match.html) which is very similar to an exhaustive switch statement.

You could write it this way, revealing the options if they exist, and doing some control flow like explicitly panicing if they don’t:
```rust, ignore, no_run
	//let options = options().unwrap();

    if let options = match options() {
        Ok(options) => options
        Err(error) => panic!(error),
    };

    println!("{} {}", options.input_path, options.output_path);
```
The Option type is actually an enum type so we lets take a full digression through enums and matching in the next section.


# error handling playground

Its worth spending some time in the option result playground here to get your mind around all this

```rust
use std::io::ErrorKind;

fn main() {
    let first_arg = Some("cat.jpg");
    let second_arg: Option<String> = None;
    let good_val: Result<u32, std::io::ErrorKind> = Ok(22);
    let definately_error: Result<u32, std::io::ErrorKind> = Err(ErrorKind::Other);

    first_arg.unwrap();
    good_val.unwrap();
    //second_arg.unwrap(); // no good

    //matching is exaustive in order
    match first_arg {
        Some(val) => println!("first_arg: {}", val),
        None => {
            // you can block scope in here and do as much as needed
            println!("third_arg");
            println!("oops")
        }
    }

    // as we've said, results are similar, just two different variants
    match definately_error {
        Ok(val) => println!("cant image how we got here: {}", val),
        Err(e) => println!("{:?}", e),
    };

    // the revealing/destructuring pattern is really handy occasionally
    if let Some(val) = first_arg {
        println!("Gotem {:?}!", val);
    }

    // theres also a ton of combinators
    if good_val.is_ok() && definately_error.is_err() {
        println!("some convoluted example here");
    }
}
```