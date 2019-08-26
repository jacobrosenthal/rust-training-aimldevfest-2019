# Vec
`std::vec::Vec` is one of the most widely used data structures in Rust programs. It's even included in the prelude (no need to `use std::vec::Vec`). Vec is a contiguous growable array container, roughly equivalent to C++'s `std::vector`.

Just like C++'s `std::vector`, `Vec` in Rust is generic over its contained type.

```rust,ignore
pub struct Vec<T> {
    ...
}
```

At the surface level, Rust generics have a similar syntax to C++ generics. However, we'll soon see that the Rust type system makes using generics much different.

For now, let's explore `Vec` a bit.

## Creating Vecs
```rust,editable
fn main() {
    let v1 = Vec::<i32>::new();
    let v2: Vec<i32> = Vec::new();
    let v3: Vec<i32> = vec![];
    let v4: Vec<i32> = vec![1, 2, 3, 4];
}
```

If the compiler can't infer the type parameter for a Vec, it will show you can error message:

```rust,editable,ignore
fn main() {
    let v = vec![1, 2, 3, 4];
}
```

`1, 2, 3, 4` are valid literals for multiple integer types. We need to annotate the type parameter somewhere.

## Vec operations
`Vec` provides familiar methods for mutation:

```rust,editable
fn main() {
    let mut v: Vec<i32> = Vec::new();

    v.push(5);
    v.push(6);
    println!("v: {:?}", v);

    let popped = v.pop();
    println!("popped: {:?}", popped);
    println!("v: {:?}", v);

    v.insert(0, 7);
    println!("v: {:?}", v);

    v.clear();
    println!("v: {:?}", v);
}
```

As we'll see in the next section on iterators, mutating a `Vec` in-place is not always the best option in Rust.

We can also grab a few properties of a Vec:

```rust,editable
fn main() {
    let v: Vec<i32> = vec![1, 2, 3, 4];
    println!("{:?}", v.len());
    println!("{:?}", v.is_empty());
}
```

# Generics
Rust generics aren't just for structs; they can be used for structs, functions, traits, and more. Let's look at a simple generic function:

TODO: Simpler generics example without the type parameter.

```rust,editable,ignore
fn generic_add<T>(a: T, b: T) -> T {
    a + b
}

fn main() {
    println!("{:?}", generic_add(1, 2))
    println!("{:?}", generic_add(1.0, 2.0))
}
```

Yup, this causes a compiler error. Unlike C++ templating, Rust generics are fully type-aware. In this case, since the type paramter `T` can be any type, how does the compiler know that the type implements the `+` operator?

In fact, there is a way we can tell the compiler we only want to allow types with the `+` operator. In Rust, the `+`, `-`, `*`, `/` and other operators are traits implemented by data types. So, if we tell the compiler that we want to only allow types that implement the `+` trait, called `Add`, it should work. So, we are constraining the type parameter to types that implement a trait.

```rust,editable
use std::ops::Add;

fn generic_add<T: Add<Output=T>>(a: T, b: T) -> T {
    a + b
}

fn main() {
    println!("{:?}", generic_add(1, 2));
    println!("{:?}", generic_add(1.0, 2.0));
}
```

You may have also noticed `Output=T`, which is what's called a type parameter on a trait. Rust doesn't assume that addition results in the same type as the operands. In this case, we constrained our function to only types that implement `Add` that results in the same type.