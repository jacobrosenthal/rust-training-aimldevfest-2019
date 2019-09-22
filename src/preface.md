# Sobel Edge Detection

Rust is a systems language pursuing the trifecta: safety, concurrency, and speed. This makes it well suited to machine learning and data science tasks. Rust experience will not be required, only existing programming experience. 

We'll spend half the session on a with an introduction to Rust and the rest using functional programming style to develop a machine learning primitive called Convolve which can be used for many tasks including edge detection, sharpening, blur and more. 

Our main goal will be the [Sobel operator](https://en.wikipedia.org/wiki/Sobel_operator). In a Sobel operator pixels in an image are 'convolved' with the Sobel kernel to produce an output that hilights edges.

![After Sobel](./images/valve_sobel.png)

[Convolution](https://en.wikipedia.org/wiki/Kernel_(image_processing)#Convolution) is an incredibly common machine learning operation and maybe you'll walk away understanding a little more about whats going on in convolutional neural networks.
