# Control Structures
So you want a branch, eh? Rust will give you most of the familiar tools, and a few improved tools.

## If
Notice, there are no required parentheses around the predicate (the compiler will gently warn you if you unnecessarily put them there). Comparison operators are nearly identical with C/C++.

[Rust operator reference](https://doc.rust-lang.org/book/appendix-02-operators.html)

```rust,editable
fn main() {
    if 1 + 1 == 2 {
        println!("It's true!");
    } else if 1 + 1 == 3 {
        println!("This world is quite odd...");
    } else {
        println!("What has the world come to?");
    }
}
```

### If is an expression!
Rust differs from C/C++ in that `if`, and other control flow structures, are also expressions that evaluate to a value. For example, the below `if` statement conditionally returns one of two strings. Notice that there are no semicolons after the return values, just like function return values.
```rust,editable
fn main() {
    println!("The world is {}",
        if 1 + 1 == 2 {
            "sane"
        } else {
            "insance"
        });
}
```

This `if` expression behaves much like the ternary operator in C/C++, but also allows you to have multiple `else if` predicates without nesting.

Control flow statements in Rust evaluate to a value, even if you are not using that value. If you end a branch with `;`, the return value will be `()`, the same as a function without a specified return value. Therefore, all the possible evaluations of the control structure need to return the same type. For example, the below code will not compile.

```rust,editable,ignore
fn main() {
    if 1 + 1 == 2 {
        println!("The world is sane.");
    } else {
        5
    }
}
```

It's quite common in Rust to see functions that look like below. The return value of the function is the evaluated value of the if expression.

```rust
fn square_if_over_10(i: i32) -> i32 {
    if i > 10 {
        i * i
    } else {
        i
    }
}
```

## Loop
Rust provides an unconditional loop construct, equivalent to `while (true) { }` in C/C++. The loop can be broken with `break` or execution can skip to the next iteration with `continue`. Unlike C/C++ loops, and just like Rust `if` statements, loops evaluate to a value as well.

```rust,editable
fn main() {
    let mut i = 2;
    let biggest = loop {
        if i > 50 {
            break i / 2;
        }
        i *= 2;
    };
    println!("The biggest power of 2 less than 50 is {}", biggest);
}
```

Just like in the `if` statements above, `break;` evaluates to the type `()`, but `break 5;` evaluates to an integer type.

## While
Rust's while loop looks familiar, minus the parentheses around the predicate.

```rust,editable
fn main() {
    let mut i = 2;
    while i < 50 {
        i *= 2;
    }
    println!("The biggest power of 2 less than 50 is {}", i / 2);
}
```

## For?
Those above contrived loop examples seem like perfect cases for a for loop right? Well, `for` is one of the places that Rust takes a familar control flow construct and completely rethinks how it should work. The below example might look more similar to Python:

```rust,editable
fn main() {
    for i in 0..10 {
        println!("{}", i);
    }
}
```

There's a good reason they look similar too! Much like Python, iterators are use all over the place. You won't find the familiar `for ( ; ; ) {}` construct, but don't worry; Rust's iterators are vastly more powerful, and safe. The next section takes us on a deep dive into iterators.




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