# external dependencies, crates.io
Lets have our code load in an image from the filesystem. Searching in the standard library for images doesn't find anything, we could take a File to binary, but lets go to the community ecosystem, [crates.io](https://www.crates.io). Searching there for images finds a crate image with ~1mil downloads which seems to be pretty popular. [image](https://crates.io/crates/image) says it wants us to add it to our Cargo.toml dependencies section so lets do that.

```toml
[dependencies]
image = "0.22.1"
```

Then in whatever file we can use this dependency:
```rust,ignore
use image;
```

The Cargo toml manifest version field is described here https://doc.rust-lang.org/cargo/reference/manifest.html#the-version-field where we learn Cargo uses [semantic versioning](https://semver.org) which allows us to version and lock dependencies at the level of risk were comfortable with. From the spec:
```text
Given a version number MAJOR.MINOR.PATCH, increment the:

MAJOR version when you make incompatible API changes,
MINOR version when you add functionality in a backwards compatible manner, and
PATCH version when you make backwards compatible bug fixes.
```

The [Cargo chapter on dependencies](https://doc.rust-lang.org/cargo/reference/specifying-dependencies.html) explains more how to do this locking. The three digit version we used above is the same as a [caret requirement](https://doc.rust-lang.org/cargo/reference/specifying-dependencies.html#caret-requirements) as if we had type `image = "^0.22.1"`. With this requirement Cargo is allowed to use any version it can satisfy between the range `>=0.22.1 <0.3.0` Semver works different below and above 1.0 with the idea that theres more breaking churn below 1.0. So for a fictional `image = "^1.2.3"` Cargo would be allowed to find patches `>=1.2.3 <2.0.0`. Refer to the spec and the book for many more clarifying examples.

The most restrictive version would be `image = "= 0.22.1` which would not allow cargo any update capability. This can be handy for to make production code reproducible. Further along that line the resolved version state of all your dependencies (recursively) is captured in the Cargo.lock file and for binaries like ours can and should be checked into the repository. This way even if you're not locking the version explicitly you're still tracking and reviewing the upstreaming of all version changes. Finally, and outside of scope here you may also use [cargo vendor](https://doc.rust-lang.org/stable/cargo/commands/cargo-vendor.html) to download all your dependencies locally and check them into your repository and or you may host your own [alternate registry](https://doc.rust-lang.org/cargo/reference/registries.html) in which you only publish vetted versions.

