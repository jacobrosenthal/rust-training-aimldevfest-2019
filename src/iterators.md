# Iterators

Iterators are one of the most powerful features in Rust! They are also a gateway drug to the functional style programming. They tend to use less memory and optimize better because the compiler can see what we're trying to do and help optimize for us. This gain in speed and reduced memory usage is really important for big datasets or constrained devices where it might be literally impossible to have the entire collection in memory at the same time.

Most collections like an array can be iterated which just means getting the collection one element at a time. This imperitive style use of an iterator should look very familiar to you. Well just print each value.

```rust,editable
fn main() {
    let array = [22.0, 1.0, 17.0];

    for i in array.iter() {
        println!("{:?}", i);
    }
}
```

One of the benefits of using iterators is all the functions we get for free. These are generally called 'combinators' but thats just a fancy name for a function that we can run against our collection. The Rust standard library provides a [large selection](https://doc.rust-lang.org/std/iter/trait.Iterator.html) of combinators for use with iterators from summing, to sorting to reversing data and much much more.

Lets reverse our array with `rev()`.

```rust,editable
fn main() {
    let array = [22.0, 1.0, 17.0];
    for i in array.iter().rev() {
        println!("{:?}", i);
    }
}
```

If we wanted to do some custom logic like add, we could certainly do it in the for loop.

```rust,editable
fn main() {
    let array = [22.0, 1.0, 17.0];

    for i in array.iter() {
        let val = i + 1.0;
        println!("{:?}", val);
    }
}
```

But thats not very functional. If we keep this up were just back at the imperitive style. Lets look at the `map` combinator instead. It lets us define a function to be run on each element one at a time, and we'll do our addition there instead. This way we can seperate concerns keeping our functions single which also has the benefit in that compiler can see better what we're doing so it can optimize better.

```rust,editable
fn main() {
    let array = [22.0, 1.0, 17.0];
    for i in array.iter().map(|i| { i + 1.0 } ) { // <- Note no semicolon, we're returning the result of our addition. Also the rust formatter will remove these uneeded brackets are needed as its only a single expression
        println!("{:?}", i);
    }
}
```

Ok now lets add AND reverse.

```rust,editable
fn main() {
    let array = [22.0, 1.0, 17.0];
    for i in array.iter().rev().map(|i| i + 1.0) {
        println!("{:?}", i);
    }
}
```

Very slick. How many times have you come across a for loop that you have to puzzle over for 10 minutes to understand what 5 things its doing at the same time? Chained combinators are very self documenting, because they all have names and can be read in order. Plus by not reimplementing simple functions, you don't reimplement the bugs either. Future you thanks present you.

Lets get even a little bit more functional. If instead of just for looping we assign the iterator to a value, we can hold the intermediate iterator and reuse it or pass it to functions.

```rust,editable
fn main() {
    let array = [22.0, 1.0, 17.0];
    let reversed_and_elevated = array.iter().rev().map(|i| i + 1.0);
    println!("{:?}", reversed_and_elevated);
}
```

Interesting. This time it didn't print our end result, but rather printed the chain of mutations plus the original data `Map { iter: Rev { iter: Iter([22.0, 1.0, 17.0]) } }`. It turns out iterators are 'lazily evaluated' which means theyre not actuall run until they're consumed. The `for in` loop and the `.sum()` are operations we have previously seen that consume an iterator. Another function that consumes an iterator is `collect()` which collects all the values back into a whole new array.  

We actually can't collect into `Array` data structures (yet). Instead we'll use a `Vec` which we won't go over much here yet, but its just another collection type that is a bit more costly and powerful than arrays.

> NOTE: This is actually the costly thing we've been avoiding all this time. It brings all the values back into memory and can take lots of memory and compute time. For debugging, desktop programming and small datasets its totally fine and eventually often you just need to consume and get back to an `Vec`. However when you move to the optimizing stage, or if you're running on constrained devices, you're looking to remove as many `collect()` as possible and just keep chaining iterators.

So lets collect our iterator and print the result. The compiler often has trouble knowing what you're trying to collect to so we'll help it out with a type hint.

```rust,editable
fn main() {
    let array = [22.0, 1.0, 17.0];
    let reversed_and_elevated = array.iter().rev().map(|i| i + 1.0);
     // We `clone` the iterator which is cheap, its not copying the datastructure just our mutation chain
    let reversed_and_elevated_array: Vec<f64> = reversed_and_elevated.clone().collect();
    println!("{:?}", reversed_and_elevated_array);
    
    // Now we can use it again or pass it to another function, or add more combinators. Note we didn't need to clone it this time since we don't use it again in this example, but we could.
    let un_reversed_and_elevated_array: Vec<f64> = reversed_and_elevated.rev().collect();  
    println!("{:?}", un_reversed_and_elevated_array);
}
```

To show what lazily evaluated means, run this example with no collect or for loop.

```rust,editable
fn main() {
    let array = [22.0, 1.0, 17.0];
    array.iter().rev().map(|i| {
        println!("{:?}", i);
        i + 1.0
    });
}
```

Note that nothing was printed because the `map()` wasn't actually run. The playground probably lost the warning, but if you do accidently run this code on your machine, the compiiler has your back with the following warning.

```text
warning: unused `Map` that must be used
 --> src/main.rs:3:5
  |
3 |     array.iter().rev().map(|i| i + 1.0);
  |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  |
  = note: `#[warn(unused_must_use)]` on by default
  = note: iterators are lazy and do nothing unless consumed

warning: 1 warning emitted
```

It gets a little more complex from here but were going to need a few more tools as we continue. `zip()` combines one value from each of two different iterators into a tuple like `(22.0, 4.0)`.

```rust,editable
fn main() {
    let array1 = [22.0, 1.0, 17.0];
    let array2 = [4.0, 19.0, 6.0];

    // you might want print in the map so you can better understand the (i,j) tuple
    let zipped = array1.iter().zip(array2.iter()).map(|(i, j)| i + j);
    let summed_array: Vec<f64> = zipped.collect();
    println!("{:?}", summed_array);
}
```

`flatten()` iterates through iterators like n dimensional structures and concatenates them one after the other, or "flattens" them. If we had a two dimensional array of arrays and wanted to turn it into a single flat array we would use `flatten()`. One more trick is needed here, our iterator is pointing at borrowed references to the arrays. We need to use `cloned()` on our iterator in order to turn our `&f32` values in `f32`. If we run this without the compiler says "value of type `Vec<f32>` cannot be built from `std::iter::Iterator<Item=&f32>"

```rust,editable
fn main() {
    let array: [[f32; 3]; 3] = [
            [1.0, 2.0, 3.0],
            [4.0, 5.0, 6.0],
            [7.0, 8.0, 9.0]
    ];
    println!("{:?}", array);

    let flat_array: Vec<f32> = array.iter().cloned().flatten().collect();
    println!("{:?}", flat_array);
}
```

Finally if we wanted to both `flatten()` and `map()` at the same time we can use `flat_map()`.

```rust,editable
fn main() {
    let array: [[f32; 3]; 3] = [
            [1.0, 2.0, 3.0],
            [4.0, 5.0, 6.0],
            [7.0, 8.0, 9.0]
        ];
    println!("{:?}", array);

    let flat_mapped_array: Vec<f32> = array.iter().flat_map(|i| {
        // each i is an entire row of the array, it hasn't been flattened yet
        println!("{:?}", i);
        // we could do something complex but well just print the inner elements
        i.iter().cloned().map(|j| {
            println!("{:?}", j); 
            j
        })
    }).collect();
    println!("{:?}", flat_mapped_array);
}
```

 Thats all we'll need for our convolution for now, but if you want even more combinators, checkout the [itertools](https://docs.rs/itertools/0.8.0/itertools/) crate.
