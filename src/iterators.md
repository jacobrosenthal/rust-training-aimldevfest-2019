# Iterators
Iterators are one of the most powerful features in Rust! They are also a gateway drug to functional programming.

The last example from the control flow section used a simple iterator, called a `Range` (similar to `range() in Python).

```rust,editable
fn main() {
    for i in 0..10 {
        println!("{}", i);
    }
}
```

## Combinators
The Rust standard library provides a large selection of combinators for use with iterators. Here's a whirlwind tour of a few important ones! You can find a full list in the [documentation for the Iterator trait](https://doc.rust-lang.org/std/iter/trait.Iterator.html).

The `map` method takes a closure to apply on each iterated element. It's the equivalent of running a given function on each element in the iterator and generating a new iterator of the return values.

```rust,editable
fn main() {
    for i in (0..10).map(|i| i * 2) {
        println!("{}", i);
    }
}
```

The `filter` method skips values that don't pass a predicate.

```rust,editable
fn main() {
    for (i, j) in (1..10).map(|i| (i, i * 3)).filter(|(i, j)| i % 2 == 0) {
        println!("{} tripled is even: {}", i, j);
    }
}
```

`flat_map` iterates through iterators and concatenates them one after the other, or "flattens" them.
```rust,editable
fn main() {
    for i in (1..5).flat_map(|i| 0..i) {
        println!("{}", i);
    }
}
```

`fold` combines all the values from an iterator, pairwise, starting with an initial value.
```rust,editable
fn factorial(n: i32) -> i32 {
    (1..n).fold(n, |x, y| x * y)
}

fn main() {
    println!("5! = {}", factorial(5));
}
```

Note: Rust actually has a combinator just for multiplicative products `product()`, and for sums: `sum()`.

```rust,editable
fn main() {
    let v: Vec<usize> = (0..5)
        .map(|i| i * 2)
        .collect();
    println!("{:?}", v);
}
```

If you want even more combinators like these, make sure to check out the [docs](https://doc.rust-lang.org/std/iter/trait.Iterator.html). And, if you want even more combinators, checkout the [itertools](https://docs.rs/itertools/0.8.0/itertools/) crate!

## Enumeration combinator
Sometimes, you miss your familar C-style for loop with its convenient access to the index; but don't run away yet, Rust has the `enumerate()` combinator for just this problem. The Iterator trait provides a combinator, called `enumerate` just for this purpose (very similar to Python's `enumerate()`).
```rust,editable
fn main() {
    for (index, value) in (100..110).enumerate() {
        println!("{}: {}", index, value);
    }
}
```

You might have noticed in the above example that `enumerate` changed the iterator elements from integers to tuples (which is why we use `for (index, value) ...`). When you chain iterators together, you can modify the type of the iterator element as you go.

It's common to see code that keeps an index or other book-keeping info along with the value of interest in a tuple:

```rust,editable
fn main() {
    for (index, value) in (100..110)
        .enumerate()
        .map(|(i, v)| (i, v * 10))
    {
        println!("{}: {}", index, value);
    }
}
```

## More complex iterators with changing element type
```rust,editable
fn main() {
    for (i, j, k) in (100..110) // type is usize
        .enumerate() // type is now (usize, usize)
        .map(|(i, j)| j * i) // type is now usize
        .map(|v| (v, v * 10, 0)) // type is now (usize, usize, usize)
    {
        println!("{}, {}, {}", i, j, k);
    }
}
```

## Thinking in iterators
You might be very familiar with this kind of nested for loop in C/C++:
```c,ignore
for (int i = 0; i < 3; i++) {
    for (int j = 0; j < 3; j++) {
        compute_something(data[i, j]);
    }
}
```

There are multiple ways to express this loop in Rust using iterators, each with its own advantages. First, we can directly translate incrementing indices into rust [Ranges](https://doc.rust-lang.org/std/ops/struct.Range.html).
```rust,editable
fn main() {
    for i in 0..3 {
        for j in 0..3 {
            println!("{}, {}", i, j);
        }
    }
}
```

Or, we can use iterator combinators to combine multiple ranges into a tuple for each element. This lets the computation code or body of the loop not care about how each element is generated. The elements of the combined iterator can be easily stored in a Vec, or broken up into chunks for parallel operation, or filtered by some predicate.
```rust,editable
fn main() {
    (0..3).for_each(|i|
        (0..3).for_each(|j| println!("{}, {}", i, j)));
}
```

As a quick preview, this could be done in parallel across CPU cores with a simple change:
```rust,editable
extern crate rayon; // just for Rust playground, not required in Rust 2018
use rayon::prelude::*;

fn main() {
    (0..3).into_par_iter()
        .for_each(|i| (0..3).for_each(|j| println!("{}, {}", i, j)));
}
```
Notice that the output order changes when we re-run the example, since the computation is distributed across multiple cores. We'll talk more about parallel iterators soon.

## Move closures
In a lot of Rust code, you never have to worry about move your local variables get captured into a closure (environment capture). However, it does seem to pop-up more often when working on complicated iterator chains. For example:

```rust,editable
fn main() {
    (0..3).flat_map(|i|
        (0..3).map(move |j| (i, j)))
        .for_each(|idx| println!("{:?}", idx));
}
```

What's that `move` keyword doing? Let's check out the compiler output without it.
```
error[E0373]: closure may outlive the current function, but it borrows `i`, which is owned by the current function
 --> src/main.rs:2:36
  |
2 |     (0..3).flat_map(|i| (0..3).map(|j| (i, j)))
  |                                    ^^^  - `i` is borrowed here
  |                                    |
  |                                    may outlive borrowed value `i`
  |
note: closure is returned here
 --> src/main.rs:2:25
  |
2 |     (0..3).flat_map(|i| (0..3).map(|j| (i, j)))
  |                         ^^^^^^^^^^^^^^^^^^^^^^
help: to force the closure to take ownership of `i` (and any other referenced variables), use the `move` keyword
  |
2 |     (0..3).flat_map(|i| (0..3).map(move |j| (i, j)))
  |                                    ^^^^^^^^
```
Basically, the compiler needs to know that the when `j` is moved into the inner closure that single-ownership is not violated. When we write `move` it indicates that the ownership is now inside the closure, and that the outer code will not use the moved values. This contrasts with the default closure capture where the variables are borrowed. In this case, borrowing doesn't work because `i` is only temporary.

For our example with inegers, it makes no difference. But, if we'ire iterating over large data structures with complicated internal state, it makes a huge difference. Lucky for us, the compiler error points out exactly what we need to do. As you write more Rust, you will start to get a feel for the `move` closure, but in the beginning, the compiler really helps point out what we need to do.