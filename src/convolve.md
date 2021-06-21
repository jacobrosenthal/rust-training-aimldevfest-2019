# Convolution Continued

Lets revisit our [Convolution operator](https://en.wikipedia.org/wiki/Kernel_(image_processing)#Convolution)

If you'll remember we left you with stubbed test for the identity kernel. When identity is run against our data it should just return the data unchanged, in the case of our test data return 5.0.

```rust ,ignore
fn convolve(kernel: &[[f32; 3]; 3], pixels: &[[f32; 3]; 3]) -> f32 {
    5.0
}
```

We can use `zip()` to combine two iterators into one iterator that yields tuple elements. Since kernel and pixels are nested arrays, `kernel.iter()` and `pixels.iter()` both give iterators over elements of type `[f32; 3]`. So, the tuple parameter `(kernel_column, input_column)` has type `([f32; 3], [f32; 3])`. We also know at the end were going to add all the products up.

```rust ,ignore
fn convolve(kernel: &[[f32; 3]; 3], pixels: &[[f32; 3]; 3]) -> f32 {
        kernel
        .iter()
        .zip(pixels.iter())
        // something has to happen here
        .sum()
}
```

**Exercise: Finish out the convolve function we stubbed earlier. You'll need everything we went through on the iterators page so head back there and think hard on how you can multiply each element of each matrix and return the sum of the results.**

<details><summary>Uncollapse for the answer. But try first!</summary>
<p>

Now in the `flat_map()` closure we iterate and zip once again and we now have our matching element from each array `(f32, f32)` which we can multiply together and then return. We've flattened our array of arrays into just numbers.

```rust ,ignore
fn convolve(kernel: &[[f32; 3]; 3], pixels: &[[f32; 3]; 3]) -> f32 {
    kernel
        .iter()
        .zip(pixels.iter())
        .flat_map(|(kernel_column, input_column)| {
            kernel_column
                .iter()
                .zip(input_column.iter())
                .map(|(k, p)| k * p)
        })
        .sum()
}
```

</p>
</details>

Ok now we have a working convolve function with an identity kernel, but we need our Sobel edge detection kernel instead now. The [Sobel Wikpedia](https://en.wikipedia.org/wiki/Sobel_operator) page shows we actually need two kernels, estimating the gradient of each Gx and Gy. In an image, the gradient describes how fast the color of the image is changing in a direction, X and Y in this case. Typically, edges change very quickly, so if we output the gradient of the image, we expect the edges to have high values.

Gx:

```text
+1 +0 -1
+2 +0 -2
+1 +0 -1
```

Gy:

```text
+1 +2 +1
+0 +0 +0
-1 -2 -1
```

and remember we usually mirror them to make our implmementation easier to reason about

```rust ,ignore
/// Kernel for the Sobel operator in the X direction
const SOBEL_KERNEL_X_MIRRORED: [[f32; 3]; 3] = [
    [-1.0, 0.0, 1.0],
    [-2.0, 0.0, 2.0],
    [-1.0, 0.0, 1.0]
];

/// Kernel for the Sobel operator in the Y direction
const SOBEL_KERNEL_Y_MIRRORED: [[f32; 3]; 3] = [
    [-1.0, -2.0, -1.0],
    [0.0, 0.0, 0.0],
    [1.0, 2.0, 1.0]
];
```

If we manually do the sobel x convolution on paper, we'd expect the result to be 8.0. Lets write another sanity test which if all is well should just pass since the hard work is already done.

```rust ,ignore
    #[test]
    fn test_convolution_sobel_x() {
        let pixels: [[f32; 3]; 3] = [[1.0, 2.0, 3.0], [4.0, 5.0, 6.0], [7.0, 8.0, 9.0]];
        let x = convolve(&SOBEL_KERNEL_X_MIRRORED, &pixels);
        assert!((x - 8.0).abs() < f32::EPSILON);
    }
```

**Exercise: Finish out a third test to make sure convolving sobel y also works correctly.**

Next up lets get back to our real image data.
