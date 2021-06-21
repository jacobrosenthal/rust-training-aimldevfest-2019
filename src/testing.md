# Convolution and testing

[Convolving a kernel with an image](https://en.wikipedia.org/wiki/Kernel_(image_processing)) is an incredibly common operation in all kinds of image processing. With it we can implement all the machine learning greatest hits, which for images as you can see includes [edge detection sharpen, blur and more](https://en.wikipedia.org/wiki/Kernel_(image_processing)#Details). Convolution should remind you of Convolution Neural nets and might give you an idea how those work too.

From wikipedia:
> Convolution is the process of adding each element of the image to its local neighbors, weighted by the kernel. This is related to a form of mathematical convolution. The matrix operation being performed—convolution—is not traditional matrix multiplication, despite being similarly denoted by *.
> For example, if we have two three-by-three matrices, the first a kernel, and the second an image piece, convolution is the process of flipping both the rows and columns of the kernel and multiplying locally similar entries and summing

```text
a b c    1 2 3
d e f  * 4 5 6
g h i    7 8 9
= (i*1) + (h*2) +  (g*3) + (f*4) + (e*5) + (d*6) + (c*7) + (b*8) + (a*9)
```

That looks a bit annoying to implement in code, but they mention an implemention flipping rows and columns of the kernel.

```text
i h g
f e d
c b a
```

Then we can just pairing up the upper left to the upper right and so on, thats our `zip()` operation.

```text
i h g    1 2 3
f e d  * 4 5 6
c b a    7 8 9
= (i*1) + (h*2) +  (g*3) + (f*4) + (e*5) + (d*6) + (c*7) + (b*8) + (a*9)
```

and we get the same result. So lets store our kernels in mirrored form.

Lets do a little test driven development in order to show off Rust's built in testing capability. So we need 2 matrices, some data (think of it as pixels of an image) and a kernel someone figured out and published for us. Did you notice the Identity Kernel on the wikipedia page? That kernel, when convolved on data, simply returns back the original data unchanged. That would be an easy test to write.

From wikipedia the identity kernel is

```text
+0 +0 +0
+0 +1 +0
+0 +0 +0
```

If we mirror in x and y, all the zeros move around and it ends up looking exactlythe same so no real work to be done there. How should we define it in Rust though? We can define the data in many structures including Vecs, Arrays, or a math library might offer a Matrix type of some kind. Looking at the identity kernel we need a 3x3 and our pixel data will be a float. For now let's use a fixed size array of fixed size arrays. This preserves the row and column structure of the kernels. We'll use the const keyword to define these as constant, static data outside of any function. The compiler will not let us in any way mutate this data.

```rust ,ignore
const IDENTITY_KERNEL_MIRRORED: [[f32; 3]; 3] = [
    [0.0, 0.0, 0.0],
    [0.0, 1.0, 0.0],
    [0.0, 0.0, 0.0]
];
```

Now we need some test data to convolve with our identity. When we convolve we're looking at the value of interest, generally the center value, but we also take with it some amount of its nearby values in order to let those effect the value we care about. This is the value were 'convolving' around. Lets just make up a fake set of pixels, 1.0 through 9.0 where the value of interest is thus `5.0`.

```rust ,ignore
let pixels: [[f32; 3]; 3] = [
    [1.0, 2.0, 3.0],
    [4.0, 5.0, 6.0],
    [7.0, 8.0, 9.0]
];
```

We need a convolve function that takes the kernel we want to convolve around the data. We could jump in and try and write the whole thing but lets procrastinate a bit more. We need our function to have 2 arguments, the kernel and the pixel data, and we probably don't want to consume them as we might want to use them in another function so lets take borrows of those as our arguments. Further if we run the convolve steps we previously discussed above by hand, you'd expect the identity, which doesnt change our value of interest, to just return the value in this case 5.0 so let's hardcode that.

```rust ,ignore
fn convolve(kernel: &[[f32; 3]; 3], pixels: &[[f32; 3]; 3]) -> f32 {
    5.0
}
```

Now we can write a test to assert that running `convolve()` indeed does return 5.0. You can simply place the testing right inside your `main.rs` file and cargo will pick it up. We can run this test with `cargo test` and we should see passing tests!

> Note rust doesn't show println data from tests, so use `cargo test -- --nocapture` to see your println debugging in this case.

> If you attempt to just assert that 5.0 and result are equal the compiler will [link you to an explanation on how to correctly compare floats](https://rust-lang.github.io/rust-clippy/master/index.html#float_cmp). Everything with floats is approximate so we can't just assert that they are equal, we have to say they're within some tolerance.

```rust ,ignore
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_convolution_identity() {
        let pixels: [[f32; 3]; 3] = [
            [1.0, 2.0, 3.0],
            [4.0, 5.0, 6.0],
            [7.0, 8.0, 9.0]
        ];
        let result = convolve(&IDENTITY_KERNEL_MIRRORED, &pixels);

        assert!((result - 5.0).abs() < f32::EPSILON);
    }
}
```

We'll come back to actually implmenting our convolve function for real in a second, but we'll need a few more tools in our toolbox so first we learn how to iterate over these arrays and multiply them using iterators.
