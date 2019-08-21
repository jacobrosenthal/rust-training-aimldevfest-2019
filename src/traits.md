# traits

But hey this seems wordy, lets just print our whole struct in one formatter.
```rust, ignore, no_run
println!("{}", options);
```

Running this results in:
```text
error[E0277]: `Opt` doesn't implement `std::fmt::Display`
  --> src/main.rs:14:20
   |
14 |     println!("{}", options);
   |                    ^^^^^^^ `Opt` cannot be formatted with the default formatter
   |
   = help: the trait `std::fmt::Display` is not implemented for `Opt`
   = note: in format strings you may be able to use `{:?}` (or {:#?} for pretty-print) instead
   = note: required by `std::fmt::Display::fmt`

error: aborting due to previous error

For more information about this error, try `rustc --explain E0277`.
error: Could not compile `training`.

To learn more, run the command again with --verbose.
```

Thats actually really rather helpful error with a several ideas for fixing it. AND in this case if say you're offline and can't google for more information it even has another page or two of content if you run the `rustc --explain E0277` command it mentions! 

> The compiler in Rust is almost always, really, actually, trying to tell you whats wrong, AND how to fix it. And if you find a case where it is not the Rust community would likely want to know how to make that error and the resulting action you should take more clear. 

So we should either use the Debug trait by changing our `{}` to `{:?}` or implement the Display trait. Let's take a moment to introduce Traits and then implement the Display trait for our Opt struct.


In Rust we stress composition over inheritence using [traits](https://doc.rust-lang.org/book/ch10-02-traits.html). Traits, much like header files, seperate the defintion from the implementation. Before we solve our Display problem by consuming someone elses trait definition, lets make a convoluted example to illustrate the syntax

We'll make a trait that has one function that println! SHOUTS our options. Traits can get the object theyre called on by using the self property which will give us access to the fields via self.input_path and self.output_path.
```rust,editable

struct Opt {
    input_path: String,
    output_path: String,
}

pub trait Shout {
    fn shout(self);
}

impl Shout for Opt {
    fn shout(self) {
        println!(
            "{} {}",
            self.input_path.to_uppercase(),
            self.output_path.to_uppercase()
        );
    }
}
```

Now anytime this trait is in namespace, which in our case it is because it defined in this same file, it is available on all instances of Opt. Lets call it.
```rust,no_run,ignore
options.shout();
```
and well should see
```text
CAT.JPG TEST.PNG
```
> The seperation of definition from implementation is incredbly powerful. This way if we make our trait public anyone downstream can customize our function for their archictecture or edge case. This keeps Rust from amongst other things passing around huge config structs full of lifecycle callbacks and other configuration overrides.

OK. So we can forget about our toy shout example. Lets implement a trait we don't own, the Display trait shipped with std Rust for our custom Opt type. As a reminder above, it said
```text
   = help: the trait `std::fmt::Display` is not implemented for `Opt`
```
Looking in the std documentation we find [Display](https://doc.rust-lang.org/std/fmt/trait.Display.html) which wants us to make a function which will be passed our instance called self, and another arg called f.

Some new syntax again:
* We finally have a function here in fmt that returns something. Thats indicated by the `->` syntax the fn definition which returns whatever type is on the right hand side. We'll cover the Result type later.
* Hopefully you're ahead of me and found the write! macro definition linked right on that page. It is just like println!  but with two big changes. Instead of printing the formatted string it sticks it in its first argument and it returns one of those Result things where println! didnt return anything.
* And wait. The write! macro function doesnt have a semicolon at the end this time! Thats indicates its return argument is implicitly being returned from this function. We could instead explicitly return and write `let blah = write!(f, "({}, {})", self.x, self.y); return blah;` but why would we.

With that info you should have what you need to finish implementing Display for Opt. And with that you've created a new type and defined and implemented traits for it, and youve implemented an trait for an external type.

> Theres one limitation with traits you will eventually find, called orphan rules. For various reasons, the compiler can't reason about traits unless you own either the trait you're implementing OR the struct you're implementing it on. Theres several ways around this including forking the underlying crate and [overriding dependencies](https://doc.rust-lang.org/cargo/reference/specifying-dependencies.html#overriding-dependencies) with Cargo patch functionality as well as the newtype pattern.
