# Rayon: embarassingly parallel, embarassingly easy
One of the big selling points of Rust is that it provides compile-time safety checking, and that extends beyond just memory to things like data races and ownership between threads.

To start using parallelism in Rust, we don't need to worry about many complicated concepts at first. We just have to know how to use iterators thanks to the [crate called "rayon"](https://docs.rs/rayon/1.1.0/rayon/)!

The rayon crate automatically handles all the details of data parallel programming for the majority of cases:

- Detects the number of CPU cores available

- Starts up a pool of worker threads

- Handles splitting the workload between workers

- Uses work stealing to rebalance the workload on-the-fly

Let's look at a simple example. Here's a vector multiply-accumulate implemented with Rust iterators:

```rust,ignore,mdbook-runnable
fn main() {
    let v1: Vec<usize> = (0..300000).collect();
    let v2: Vec<usize> = (300000..600000).collect();
    let sum: usize = v1.iter().zip(v2.iter()).map(|(a, b)| a * b).sum();
    println!("Sum: {}", sum);
}
```

And now with data parallelism across all the cores available:

```rust,ignore,mdbook-runnable
#extern crate rayon;
use rayon::prelude::*;

fn main() {
    let v1: Vec<usize> = (0..300000).collect();
    let v2: Vec<usize> = (300000..600000).collect();
    let sum: usize = v1.par_iter().zip(v2.par_iter()).map(|(a, b)| a * b).sum();
    println!("Sum: {}", sum);
}
```

## Using rayon in the Sobel filter
We can create an iterator that yields every pixel (X, Y) and `par_iter()` on that.

```rust,ignore
fn sobel_filter(input: &GrayImage) -> GrayImage {
    let mut result = input.sub_image(1, 1, input.width() - 2, input.height() - 2)
        .to_image();

    let pixels_iter = (1..(image.width() - 1))
        .flat_map(|x| (1..(image.height() - 1)).map(|y| (x, y)));

    pixels_iter.into_par_iter().for_each(|(x, y) {
        let pixels = [
            [input.get_float_luma(x - 1, y - 1),
                input.get_float_luma(x - 1, y),
                input.get_float_luma(x - 1, y + 1)],
            [input.get_float_luma(x, y - 1),
                input.get_float_luma(x, y),
                input.get_float_luma(x, y + 1)],
            [input.get_float_luma(x + 1, y - 1),
                input.get_float_luma(x + 1, y),
                input.get_float_luma(x + 1, y + 1)]];

        let gradient_x = convolve(&SOBEL_KERNEL_X, &pixels);
        let gradient_y = convolve(&SOBEL_KERNEL_Y, &pixels);
        let magnitude = (gradient_x.powi(2) + gradient_y.powi(2)).sqrt();
        result.put_float_luma(x - 1, y - 1, magnitude);
    });

    result
}
```

Looks like we have a problem though! The mutable result image cannot be mutably borrowed in more than one place! The Rust compiler detects that we are attemping to share a mutable resource between multiple threads.

One way we can get around this is to collect the convolution results (the computationally expensive part) and then apply the results in a single thread after.

```rust,ignore
fn sobel_filter(input: &GrayImage) -> GrayImage {
    let mut result = input.sub_image(1, 1, input.width() - 2, input.height() - 2)
        .to_image();

    let pixels_iter = (1..(image.width() - 1))
        .flat_map(|x| (1..(image.height() - 1)).map(|y| (x, y)));

    let convolved_pixels: Vec<(u32, u32, f32) = pixels_iter.into_par_iter()
        .map(|(x, y)| {
            let pixels = [
                [input.get_float_luma(x - 1, y - 1),
                 input.get_float_luma(x - 1, y),
                 input.get_float_luma(x - 1, y + 1)],
                [input.get_float_luma(x, y - 1),
                 input.get_float_luma(x, y),
                 input.get_float_luma(x, y + 1)],
                [input.get_float_luma(x + 1, y - 1),
                 input.get_float_luma(x + 1, y),
                 input.get_float_luma(x + 1, y + 1)]];

            let gradient_x = convolve(&SOBEL_KERNEL_X, &pixels);
            let gradient_y = convolve(&SOBEL_KERNEL_Y, &pixels);
            let magnitude = (gradient_x.powi(2) + gradient_y.powi(2)).sqrt();
            (x, y, magnitude)
    }).collect();

    for (x, y, magnitude) in convolved_pixels.iter() {
        result.put_float_luma(x - 1, y - 1, magnitude);
    }

    result
}
```

## Granularity
The parallelized version above compiles and runs fine. But if we were to open it up in a profiler, we'd see overhead due to the fine-grain parallelism we applied. It's usually best to try to "chunk" work up.

Since we're processing an image, we can easily chunk the work by column or row.

```rust,ignore
fn sobel_filter(input: &GrayImage) -> GrayImage {
    let mut result = input.sub_image(1, 1, input.width() - 2, input.height() - 2)
        .to_image();

    let convolved_pixels: Vec<(u32, u32, f32)> = (1..(image.width() - 1))
        .into_par_iter()
        .flat_map(|x| {
            (1..(image.height() - 1)).map(|y| {
                let pixels = [
                    [input.get_float_luma(x - 1, y - 1),
                    input.get_float_luma(x - 1, y),
                    input.get_float_luma(x - 1, y + 1)],
                    [input.get_float_luma(x, y - 1),
                    input.get_float_luma(x, y),
                    input.get_float_luma(x, y + 1)],
                    [input.get_float_luma(x + 1, y - 1),
                    input.get_float_luma(x + 1, y),
                    input.get_float_luma(x + 1, y + 1)]];

                let gradient_x = convolve(&SOBEL_KERNEL_X, &pixels);
                let gradient_y = convolve(&SOBEL_KERNEL_Y, &pixels);
                let magnitude = (gradient_x.powi(2) + gradient_y.powi(2)).sqrt();
                (x, y, magnitude)
            })
        }).collect();

    for (x, y, magnitude) in convolved_pixels.iter() {
        result.put_float_luma(x - 1, y - 1, magnitude);
    }

    result
}
```