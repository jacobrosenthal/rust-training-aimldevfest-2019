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

If you want even more combinators, make sure to check out the [docs](https://doc.rust-lang.org/std/iter/trait.Iterator.html). And, if you want even more combinators, checkout the [itertools](https://docs.rs/itertools/0.8.0/itertools/) crate!

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