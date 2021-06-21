# Image Processing Example

## Getting the pixels

Our convolution function is ready, but we are missing the connection between the image we converted to luma and the convolution operator. Let's look into the [docs on GrayImage](https://docs.rs/image/0.22.1/image/type.GrayImage.html) to see how we can get pixel values out.

![GrayImage docs](./images/gray-image-docs.png)

At first glance, it doesn't look like there are many methods, huh?

Let's take a closer look. `GrayImage` is defined as a type alias of a specific variation of ImageBuffer (using generic type parameters). If we click on `ImageBuffer` (usually in Rust docs, you can click on a type name to see its docs), we will see the [full list of available methods](https://docs.rs/image/0.22.1/image/struct.ImageBuffer.html).

![get_pixel docs](./images/image-get-pixel-docs.png)

There's a `get_pixel` method! Oh, but the return type is `&P`, that's weird. If we look at the declaration of ImageBuffer though, we see that `P` must implement the `Pixel` trait. And if we look at the `Pixel` [trait docs](https://docs.rs/image/0.22.1/image/trait.Pixel.html), we see a method called `channels()` that gives us a slice of the pixel's values, one for each channel. Since out image is grayscale (luma), we expect just one channel.

This might seem over-complicated. However, by abstracting away the underlying storage formats, the "image" crate lets users build processing systems that are general over many image formats. Remember, the Rust compiler boils down all of the abstractions into highly optimized code. So we can have our generics and safety while writing high-performance code!

For our case, we just have a GrayImage with pixels of type `Luma<u8>` that implement the `Pixel` trait. So we should be able to fetch a pixel pretty easily. Here's a go:

```rust ,ignore
use image::Pixel; // trait for '.channels()'

let input_image = image::open(&arguments.input_path)
    .expect("Failed to open input image file");

let input_image = input_image.to_luma();

println!("Pixel 0, 0: {}", input_image.get_pixel(0, 0).channels()[0]);

input_image.save(&arguments.output_path)
    .expect("Failed to save output image to file");
```

Generally, well-written Rust crates provide comprehensive types like this to cover the data formats and structures that they operate on. An image library in C/C++ may provide a raw buffer of pixels, which is easy to access. But, as soon as you have to deal with multiple formats, multiple pixel orderings (RGB, BGR, RGBA, etc.), it can be difficult to ensure all code branches are correct. With Rust, the type system will catch these errors at compile time.

Now that we can grab pixels, let's write a function that takes the pixel values and calls our convolution function. First we'll start with this signature, and copying the input. We need a place to store the resulting convolved pixel values, and we want an image of the same dimensions and data types. `clone()` is an easy way to get that. Notice that `result` is declared as `mut` since we will be modifying its contents.

```rust ,ignore
use image::{GrayImage, Pixel};

fn sobel_filter(input: &GrayImage) -> GrayImage {
    let mut result = input.clone();

    result
}
```

To start with, let's just create the block of pixels to feed the convolution for each center pixel.

```rust ,ignore
use image::{GrayImage, Pixel};

fn sobel_filter(input: &GrayImage) -> GrayImage {
    let mut result = input.clone();

    for x in 0..input.width() {
        for y in 0..input.height() {
            let pixels = [
                [
                    input.get_pixel(x - 1, y - 1).channels()[0],
                    input.get_pixel(x - 1, y).channels()[0],
                    input.get_pixel(x - 1, y + 1).channels()[0],
                ],
                [
                    input.get_pixel(x, y - 1).channels()[0],
                    input.get_pixel(x, y).channels()[0],
                    input.get_pixel(x, y + 1).channels()[0],
                ],
                [
                    input.get_pixel(x + 1, y - 1).channels()[0],
                    input.get_pixel(x + 1, y).channels()[0],
                    input.get_pixel(x + 1, y + 1).channels()[0],
                ],
            ];
        }
    }

    result
}
```

We'll need to throw a call into `fn main()` to use this.

```rust ,ignore
let input_image = image::open(&arguments.input_path)
    .expect("Failed to open input image file");

let input_image = input_image.to_luma();

let input_image = sobel_filter(&input_image);

input_image.save(&arguments.output_path)
    .expect("Failed to save output image to file");
```

And then when we run this... what happened?!? What does it mean we attempted subtraction with overflow?

Well, in Rust debug builds, the primitive integer types are checked for overflows an underflows in the basic operations. Don't worry, these are not enabled in the release build unless you specifically want.

And, just like now, the overflow checks in debug builds help catch bugs early on.

## Handling the edges

The overflow is happening because of the `x - 1` and `y - 1` when x or y is zero. Remember, the kernel includes one pixel left, right, up and down from the one its currently operating on. When were on the far border of our image that pixel doesn't exist. This is indicative of a bigger question: how should we handle the edges of the image?

As the [Wikipedia page on convolution kernels](https://en.wikipedia.org/wiki/Kernel_(image_processing)#Edge_Handling) explains, there are several ways:

- Extend the image by duplicating pixels at the edge
- Wrap around to the other side
- Crop the output image 2 pixels smaller in X and Y
- Crop the kernel on the edges and corners

If we crop the output image, we can easily adapt our code. The ImageBuffer struct implements the GenericImage trait which has a function called `sub_image` that gives us a view into rectangular section of an image. With a `SubImage` we can call `to_image()` to get a cropped `ImageBuffer` back out.

```rust ,ignore
use image::{GenericImage, GrayImage, Pixel};

fn sobel_filter(input: &GrayImage) -> GrayImage {
    let mut result = input
        .sub_image(1, 1, input.width() - 2, input.height() - 2)
        .to_image();

    //start convolve in 1 pixel
    for x in 1..(input.width() - 1) {
        //start convolve in 1 pixel
        for y in 1..(input.height() - 1) {
            let pixels = [
                [
                    input.get_pixel(x - 1, y - 1).channels()[0],
                    input.get_pixel(x - 1, y).channels()[0],
                    input.get_pixel(x - 1, y + 1).channels()[0],
                ],
                [
                    input.get_pixel(x, y - 1).channels()[0],
                    input.get_pixel(x, y).channels()[0],
                    input.get_pixel(x, y + 1).channels()[0],
                ],
                [
                    input.get_pixel(x + 1, y - 1).channels()[0],
                    input.get_pixel(x + 1, y).channels()[0],
                    input.get_pixel(x + 1, y + 1).channels()[0],
                ],
            ];
        }
    }

    result
}
```

Oh, and did you find a place where clone might be handy?. Cool. No more overflows. We should get the convolution in there! Usually, we also divide by a constant value to "normalize" the result (really just make sure it is within the 0.0-1.0 range). For the Sobel operator on a 3x3 block of pixels, a divisor of 8.0 works well.

```rust ,ignore
use image::{GenericImage, GrayImage, Pixel};

fn sobel_filter(input: &GrayImage) -> GrayImage {
    let mut result = input
        .clone()
        .sub_image(1, 1, input.width() - 2, input.height() - 2)
        .to_image();

    //start convolve in 1 pixel
    for x in 1..(input.width() - 1) {
        //start convolve in 1 pixel
        for y in 1..(input.height() - 1) {
            let pixels = [
                [
                    input.get_pixel(x - 1, y - 1).channels()[0],
                    input.get_pixel(x - 1, y).channels()[0],
                    input.get_pixel(x - 1, y + 1).channels()[0],
                ],
                [
                    input.get_pixel(x, y - 1).channels()[0],
                    input.get_pixel(x, y).channels()[0],
                    input.get_pixel(x, y + 1).channels()[0],
                ],
                [
                    input.get_pixel(x + 1, y - 1).channels()[0],
                    input.get_pixel(x + 1, y).channels()[0],
                    input.get_pixel(x + 1, y + 1).channels()[0],
                ],
            ];

            // normalize divisor of 8.0 for Sobel
            let gradient_x = convolve(&SOBEL_KERNEL_X, &pixels) / 8.0;
            let gradient_y = convolve(&SOBEL_KERNEL_Y, &pixels) / 8.0;
        }
    }

    result
}
```

Uh oh. Now we have a different problem. Our `GrayImage` gives us `u8` from `get_pixel(x, y).channels()[0]`, but `convolve()` expects the pixels to be f32.
We can explicitly cast that with 'as f32'.

We can also add a couple lines to combine our two kernels into a single magnitude with the sum of squares. Then well need to turn our resulting f32 into a u8 Luma type before storing it back into the resulting image.

```rust ,ignore
use image::{GenericImage, GrayImage, Luma, Pixel};

fn sobel_filter(input: &GrayImage) -> GrayImage {
    let mut result = input
        .clone()
        .sub_image(1, 1, input.width() - 2, input.height() - 2)
        .to_image();

    //start convolve in 1 pixel
    for x in 1..(input.width() - 1) {
        //start convolve in 1 pixel
        for y in 1..input.height() - 1 {
            let pixels = [
                [
                    input.get_pixel(x - 1, y - 1).channels()[0] as f32,
                    input.get_pixel(x - 1, y).channels()[0] as f32,
                    input.get_pixel(x - 1, y + 1).channels()[0] as f32,
                ],
                [
                    input.get_pixel(x, y - 1).channels()[0] as f32,
                    input.get_pixel(x, y).channels()[0] as f32,
                    input.get_pixel(x, y + 1).channels()[0] as f32,
                ],
                [
                    input.get_pixel(x + 1, y - 1).channels()[0] as f32,
                    input.get_pixel(x + 1, y).channels()[0] as f32,
                    input.get_pixel(x + 1, y + 1).channels()[0] as f32,
                ],
            ];

            // normalize divisor of 8.0 for Sobel
            let gradient_x = convolve(&SOBEL_KERNEL_X_MIRRORED, &pixels) / 8.0;
            let gradient_y = convolve(&SOBEL_KERNEL_Y_MIRRORED, &pixels) / 8.0;
            let magnitude = (gradient_x.powi(2) + gradient_y.powi(2)).sqrt();
            //place our pixel off by one because of crop
            result.put_pixel(x - 1, y - 1, Luma([(magnitude) as u8]));
        }
    }

    result
}
```

Now if we `cargo run`, the output should be interesting. If the runtime is a bit long, you might try `cargo run --release`. Running in release mode can make a massive difference.

## Sucess

![Result image](./images/valve_sobel.png)

## Extra credit

- Can you implement edge extension instead of cropping?
- Can you implement a box blur instead of the Sobel operator?
- Can you extend the command line interface to allow the user to select what filter to apply?
