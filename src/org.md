# Code Organization & Modules
At this point, we're starting to pollute one Rust source file with a few unrelated operations and imports. Rust makes it pretty easy to refactor code into a hierarchy of modules, and sprinkle in encapsulation where appropriate.

## What's a module?
A module is very simliar to a C++ namespace, in that it is a named scope containing declarations of structs, enums, functions, traits, etc.

Let's take a quick look:

```rust,editable
mod say {
    pub fn hello() {
        println!("I'm a module");
    }
}

fn main() {
    say::hello();
}
```

## Visibility

In the simple example above, we also see `pub` which is a visibility specifier. By default, everything is Rust is visible within the same module, and its descendents. If we want to use a declaration outside of its module, we need to declare it as `pub`.

Rust also gives you some more tools for fine-grain visibility control:

- `pub(super)`: visible to containing module
- `pub(crate)`: visible to whole containing crate
- `pub(some::path::here)`: visible in the specified module namespace

## Ways to make a module
1. The `mod {}` syntax above

2. As a separate file

```ignore
crate
- Cargo.toml
- src
  - lib.rs (or main.rs)
  - mymodule.rs
```

In lib.rs (or main.rs)
```rust,ignore
mod mymodule;
```

In mymodule.rs:
```rust,ignore
pub fn myfunction() {
    ...
}
```

3. As a directory (for when your module has modules)

```ignore
crate
- Cargo.toml
- src
  - lib.rs (or main.rs)
  - bigmodule
    - mod.rs
    - submodule.rs
```

In lib.rs (or main.rs):
```rust,ignore
mod bigmodule;
```

In mod.rs:
```rust,ignore
mod submodule;

fn function_in_bigmodule() {
    ...
}
```

In submodule.rs
```rust,ignore
fn function_in_submodule() {
    ...
}
```

## Let's refactor the Sobel filter program
We can refactor the sobel filter function, convolution function, and kernels into separate modules.

This way, the main module is only concerned with user input and calling out to the other modules to execute.

## Re-exporting
Rust also includes a mechanism for re-exporting imported modules, functions, structs, etc from within a module.

For example, our Sobel filter module could re-export `GrayImage` since all callers will need to use it.

```rust,ignore
pub use image::GrayImage;
```

You can `pub use` crates, whole modules, individual functions, or even sets of things (`pub use some_crate::{thing1, thing2};`).

## Custom preludes
You probably saw in the previous chapters that to import rayon we used `use rayon::prelude::*`.

This a common pattern in Rust crates to create an easy way to import a group of functions, traits, etc that are commonly all used together. For example, the standard library also uses this pattern for `std::io::prelude::*`, which includes most functions, traits, and structs necessary for file I/O.

If we take a look at the [rayon docs](https://docs.rs/rayon/1.1.0/rayon/prelude/index.html), you'll see exactly this pattern.

You create a prelude by creating a module, just like other modules. However, typically prelude modules consist solely of `pub use` statements.

```rust,ignore
mod prelude {
    pub use sobel::sobel_filter;
    pub use image::GrayImage;
}
```