# Testing


[Convolving a kernel with an image](https://en.wikipedia.org/wiki/Kernel_(image_processing)) is an incredibly common operation in all kinds of image processing. With it we can implement all the machine learning greatest hits, which for images as you can see includes [edge detection sharpen, blur and more](https://en.wikipedia.org/wiki/Kernel_(image_processing)#Details). Convolution should remind you of Convolution Neural nets and might give you an idea how those work too.

Did you notice the Identity Kernel on the page? That kernel returns back the result unchanged. That would be a helpful test. We can define the kernels multiple ways, but for now let's use a fixed size array of fixed size arrays. This preserves the row and column structure of the kernels. We'll use the const keyword to define these as constant, static data outside of any function. The compiler will not let us in any way mutate this data.

```rust,ignore

const IDENTITY: [[f32; 3]; 3] = [
    [0.0, 0.0, 0.0],
    [0.0, 1.0, 0.0],
    [0.0, 0.0, 0.0]
];

```

Lets just make up a fake set of pixels, 1.0 through 9.0. The convolve process for our test data and identity would start something like this

That may sound complicated, but it boils down to this: multiply each pixel in the block (matrix A of pixel values) with the corresponding value in the kernel matrix, and then add up all the results.

![convolution process](./images/convolution.png)

And we can see that the result will be 5 when where all done.

We all know if it compiles it ships. But let's write a quick sanity test for our convolution function too. Typically, tests are separated into a "tests" module, but kept inline with the code they verify. So, in our main file we can create a module, and mark it so that it is only compiled in the test configuration. Any function marked with `#[test]` will be run as a test.
```rust,ignore
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_convolution_identity() {
		let pixels: [[f32; 3]; 3] = 
		    [1.0, 2.0, 3.0],
		    [4.0, 5.0, 6.0],
		    [7.0, 8.0, 9.0]
		];
        assert_eq!(convolve(&IDENTITY, &pixels), 5.0);
    }
}
```


Now lets stub out our convolve function. Notice that both the kernel and pixels are borrowed, not moved, since the kernel will be re-used for all pixels. At least while starting out in Rust, prefer borrowing to moving unless you have a good reason. Lets hardcode our expected result of 5 for now.

```rust, ignore
fn convolve(kernel: &[[f32; 3]; 3], pixels: &[[f32; 3]; 3]) -> f32 {
    5
}
```
Run the tests with `cargo test` and we should see passing tests.

Next up well learn how to iterate over these arrays and multiply them using iterators.

