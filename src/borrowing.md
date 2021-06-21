# Borrowing

The borrow checker is probably Rust's most distinctive feature. To enable zero cost abstractions, Rust does not have a garbage collector. However, Rust also doesn't rely on explicit calls to `free()` like C. Instead Rust enforces "ownership" for all memory objects. The rules of the ownership system are pretty simple:

1. There is only ever one owner of a memory object at a time (struct, enum, primitive, etc)

2. Immutable (read-only) ownership can be borrowed multiple places simultaneously

3. Mutable (writable) ownership can only be borrowed once at a time and exclusively

4. An object must live at least as long as all of its borrows

## Rule #1: single owner

```rust ,editable,ignore,mdbook-runnable
fn eat(s: String) {
    println!("Eating {}", s);
}

fn main() {
    let food = String::from("salad");
    eat(food);
    eat(food);
}
```

The compiler error tells us exactly what's wrong. The `fn eat(s: String)` signature says that `s` will be moved into the function upon calling. In other words, the function `eat` will take ownership of `s`. Unless we pass ownership back to the caller, ownership will remain there. This is called "consuming" a parameter.

Here's how we can pass ownership back.

```rust ,editable
fn eat(s: String) -> String {
    println!("Eating {}", s);
    s
}

fn main() {
    let food = String::from("salad");
    let owned_food = eat(food);
    eat(owned_food);
}
```

## Rule #2: multiple immutable borrows

If we change the function signature to borrow `s` instead, the problem goes away.

```rust ,editable
fn stare_at(s: &String) {
    println!("Drooling over {}", s);
}

fn main() {
    let food = String::from("donut");
    let another_ref = &food;
    stare_at(&food);
    stare_at(&food);
    stare_at(another_ref);
}
```

## Rule #3: mutable borrows are exclusive

Only a mutable borrow for an object can exist at a time. This prevents many subtle errors where internal state is mutated while other does not expect it. In C++, modifying a container while iterating through it is a classic example.

```rust ,editable,ignore,mdbook-runnable
fn main() {
    let mut number: usize = 32;

    let borrowed = &number;
    println!("Borrowed: {}", borrowed);

    let mut_borrowed = &mut number;
    *mut_borrowed = 59;
    println!("Mut borrowed: {}", mut_borrowed);
    println!("Borrowed: {}", borrowed);
}
```

You'll notice the compiler gave us an error because we have an immutable borrow out there when we try to mutably borrow number. Any additional borrows, mutable or not, will make a mutable borrow invalid.

When we have an exclusive mutable borrow, all is good.

```rust ,editable
fn main() {
    let mut number: usize = 32;

    let mut_borrowed = &mut number;
    *mut_borrowed = 59;
    println!("Mut borrowed: {}", mut_borrowed);
}
```

## Rule #4: lifetime >= borrow time

In the example below, we borrow a temporary value inside the if statement branches. The temporary value does not last beyond the if statement branch, so the compiler tells us that our borrow is invalid. We can't borrow an object that doesn't exist.

```rust ,editable,ignore,mdbook-runnable
fn main() {
    let borrowed = if 1 + 1 == 2 {
        let msg = "The world is sane.";
        &msg
    } else {
        let msg = "The world is insane!";
        &msg
    };
}
```

## The learning curve

Many new Rustaceans report that the fighting the borrow checker is the hardest part of learning Rust, and kind of like hitting a wall. Programmers coming from C/C++ tend to have a hard time because they know exactly what they want to do, but the Rust compiler "won't let them do it".

Over time, the borrowing rules and working with the borrow checker become second nature. In fact, the borrow checker enforces rules that well-written C++ code should abide by anyway. Working with the borrow checker is kind of like pair programming with a memory ownership expert.

The borrowing rules prevent all kinds of common C++ memory and security errors. For example, you can't create a dangling borrow, the compiler won't let you. In C/C++, you can quite easily create a dangling pointer!

## Cloning

While you are learning Rust, you will face another temptation: clone everything! The `Clone` trait in Rust provides the method `clone()` which creates a copy of any objects that implements `Clone`. When something is cloned, the borrows on the original do not apply to the new copy.

```rust ,editable
fn main() {
    let mut number: usize = 32;

    let cloned = number.clone();
    println!("Cloned: {}", cloned);

    let mut_borrowed = &mut number;
    *mut_borrowed = 59;
    println!("Mut borrowed: {}", mut_borrowed);
    println!("Cloned: {}", cloned);
}
```

## Lifetimes and scopes

One last thing to note about lifetimes is that they are tied to scopes. So a borrow must exist in a scope at or below the level of the ownership.

```rust ,editable,ignore,mdbook-runnable
fn main() {
    let mut number: usize = 32;
    {
        let borrowed = &number; // works!
        println!("It works: {}", borrowed);
    }

    {
        let number2: usize = 64;
    }
    let borrowed2 = &number2; // fails!
}
```

Also, Rust allows a scope to return a value. This is useful for temporarily borrowing a value in a limited scope and computing some value without creating a whole separate function for it.

```rust ,editable
fn main() {
    let number: usize = 32;
    let new_number = {
        let borrowed = &number;
        borrowed + 16
    };
    println!("{}", new_number);
}
```

Prior to Rust 2018 edition, it used to be common to use scopes to explicitly end borrows. The below code shows how we can use an extra scope (curly braces) to end a borrow early to allow a mutable borrow. With Rust 2018, the compiler is actually smart enough to detect this on its own, so we don't worry about it much unless you have a specific case the compiler can't figure out.

```rust ,editable
fn main() {
    let mut number: usize = 32;

    {
        let borrowed = &number;
        println!("Borrowed: {}", borrowed);
    } // borrow ends here

    // no living borrows, so &mut is ok!
    let mut_borrowed = &mut number;
    *mut_borrowed = 59;
    println!("Mut borrowed: {}", mut_borrowed);
}
```
