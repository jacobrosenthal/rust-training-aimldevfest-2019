# Iterators


Iterators are one of the most powerful features in Rust! They are also a gateway drug to functional programming.

A lot of collections like array have a `.iter()` method available so you can get the collection as an iterator to operate on each element one at at time.

```rust,editable
fn main() {
    let array = [3.0, 2.0, 1.0];
    let iter = array.iter();
    println!("{:?}", iter);
}
```

This allows you to deal with values one at a time. This is really important for big datasets or small constrained devices where it might be literally impossible to have the entire collection in memory at the same time.

Lets get the `sum()` of our array
```rust,editable
fn main() {
    let array = [3.0, 2.0, 1.0];
    let iter = array.iter();
    let total: f32 = iter.sum();
    println!("{:?}", total);
}
```

The Rust standard library provides a [large selection](https://doc.rust-lang.org/std/iter/trait.Iterator.html) of combinators for use with iterators. Theres one called `map()` that lets you define a function (closure actually) to visit each element and mutate it before returning it.

But wait.. why this doesn't work?

```rust,no_run
fn main() {
    let array = [3.0, 2.0, 1.0];
    let iter = array.iter();
    iter.map(|val| {
        println!("{:?}", val);
        val + 1.0
    });
}
```
What do you mean its unused?!
```text
warning: unused `std::iter::Map` that must be used
  --> src/main.rs:85:5
   |
85 | /     iter.map(|val| {
86 | |         println!("{:?}", val);
87 | |         val + 1.0
88 | |     });
   | |_______^
   |
   = note: #[warn(unused_must_use)] on by default
   = note: iterators are lazy and do nothing unless consumed
```

It turns out `println!()` and `sum()` both consume iterators so we got lucky up until now. `map()` just visits each. Someone needs to pull our values through. Another consumer is our good old `for in` construct. Lets use that:

```rust,editable
fn main() {
    let array = [3.0, 2.0, 1.0];

    for i in array.iter().map(|val| {
        println!("{:?}", val);
        val + 1.0
    }) {
        println!("{:?}", i);
    }
}
```

> Note this is a great debugging strategy. If you get lost, throw (non mutating) map in the middle of you combinator chains to do a little print debugging.

It gets more complex from here. `zip()` combines a value from two different iterators into a tuple.

```rust,editable
fn main() {
    let array1 = [3.0, 2.0, 1.0];
    let array2 = [4.0, 5.0, 6.0];

    for i in array1.iter().zip(array2.iter()) {
        println!("{:?}", i);
    }
}
```

`flat_map` iterates through iterators like n dimensional structures and concatenates them one after the other, or "flattens" them.
```rust,editable
fn main() {

    let array: [[f32; 3]; 3] = [
            [1.0, 2.0, 3.0],
            [4.0, 5.0, 6.0],
            [7.0, 8.0, 9.0]
        ];

    //flat_map gives a closure where we could transform, instead we just return it
    for i in array.iter().flat_map(|j| j) {
        println!("{:?}", i);
    }
}
```

If you want even more combinators like these, make sure to check out the [docs](https://doc.rust-lang.org/std/iter/trait.Iterator.html). And, if you want even more combinators, checkout the [itertools](https://docs.rs/itertools/0.8.0/itertools/) crate!
