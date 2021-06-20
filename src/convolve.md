# Convolution

With what you now know about iterators, lets revisit our [Convolution operator](https://en.wikipedia.org/wiki/Kernel_(image_processing)#Convolution)

If you'll remember we left you with stubbed test

```rust, ignore
fn convolve(kernel: &[[f32; 3]; 3], pixels: &[[f32; 3]; 3]) -> f32 {
    5.0
}
```

**Exercise: Finish out the convolve function we stubbed earlier. You'll need everything we went through on the iterators page so head back there and think hard on how you can create an iterator from both pixels and kernel, multiply each element of each matrix and return the sum of the results.**

> Note rust doesn't show println data from tests, so use `cargo test -- --nocapture` to see your println debugging in this case.

<details><summary>Uncollapse for a hint. But try first!</summary>
<p>

Here we use `zip` to combine two iterators into one iterator that yields tuple elements. Since kernel and pixels are nested arrays, `kernel.iter()` and `pixels.iter()` both give iterators over elements of type `[f32; 3]`. So, the tuple parameter `(kernel_col, input_col)` has type `([f32; 3], [f32; 3])`. Therefore in the closure, we iterate and zip once again, to yield elements of type `(f32, f32)` that we can multiply together. Finally we use the sum combinator to add all the products up.
<details><summary>Uncollapse for the answer. But try first!</summary>
<p>

```rust,ignore
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
```

</p>
</details>

</p>
</details>

Ok now we have a working convolve function with an identity kernel, but we need our Sobel edge detection kernel instead now. The [Sobel Wikpedia](https://en.wikipedia.org/wiki/Sobel_operator) page shows we actually need two kernels, one each estimating the gradient ofs Gx and Gy. In an image, the gradient describes how fast the color of the image is changing in a direction, X and Y in this case. Typically, edges change very quickly, so if we output the gradient of the image, we expect the edges to have high values.

Copy in the kernel definitions to your code

```rust,ignore
/// Kernel for the Sobel operator in the X direction
const SOBEL_KERNEL_X: [[f32; 3]; 3] = [
    [-1.0, -2.0, -1.0],
    [0.0, 0.0, 0.0],
    [1.0, 2.0, 1.0]
];

/// Kernel for the Sobel operator in the Y direction
const SOBEL_KERNEL_Y: [[f32; 3]; 3] = [
    [-1.0, 0.0, 1.0],
    [-2.0, 0.0, 2.0],
    [-1.0, 0.0, 1.0]
];
```

And write another sanity test which if all is well should just pass!

```rust,ignore
    #[test]
    fn test_convolution_sobel() {
        let pixels: [[f32; 3]; 3] = [
            [1.0, 2.0, 3.0],
            [4.0, 5.0, 6.0],
            [7.0, 8.0, 9.0]
        ];
        assert_eq!(convolve(&SOBEL_KERNEL_X, &pixels), 24.0);
        assert_eq!(convolve(&SOBEL_KERNEL_Y, &pixels), 8.0);
    }
}
```

Next up lets get back to our real image data.
