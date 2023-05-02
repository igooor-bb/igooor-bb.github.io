---
layout: post
author: Igor Belov
title: Calculating Histograms in Swift using Accelerate Framework
---

In this article, you will learn how to calculate histograms for an image in Swift using the Accelerate framework. The Accelerate framework is a powerful library that provides high-performance operations for image processing, linear algebra, and other mathematical functions. The provided code will be explained step by step, so by the end of the article, you will have a good understanding of how to use the Accelerate framework to calculate image histograms.

<!-- more -->

## What is histogram?

In simple terms, a histogram is a way of visualizing the distribution of values in a set of data. When it comes to image processing, a histogram illustrates the number of image pixels with specific intensity levels. The horizontal axis of the histogram shows intensity values, which usually range from 0 to 255, while the vertical axis indicates the quantity of pixels with that specific intensity level.

What does this mean in real-life scenarios? Imagine you have a black and white image featuring many dark areas. Examining the histogram of that image, you'll notice a peak on the graph's left side, revealing that a large number of pixels possess low intensity values. Conversely, if you have a bright, high-contrast image, the histogram will display peaks at both ends of the graph, signifying numerous pixels with both low and high intensity values.

Moreover, histograms help in assessing an image's color balance. For a color image, the histogram will present separate charts for each color channel (red, green, and blue). By examining the histograms of each color channel, you can determine if a specific color dominates the image or if the colors are evenly distributed.

## Coding

Before you begin, you'll need to import the Accelerate framework by including the following line at the start of your Swift file:

```swift
import Accelerate
```

Then, create a function named `calculateHistogram` that accepts an input image of the `CGImage` type and returns a 2D array of histograms for each color channel (red, green, blue, and alpha).

```swift
func calculateHistogram(image: CGImage) -> [[vImagePixelCount]] {
    // Proceed with the next step here...
}
```

The `vImagePixelCount` type is a 32-bit unsigned integer that represents pixel counts in a histogram.

Next, create a `vImage_CGImageFormat` object to specify the format of the input image. This encompasses the number of bits per component (usually 8), the total bits per pixel (typically 32, since we have 8 bits for each color channel and 8 bits for alpha), the color space (using the device RGB color space), the bitmap info (setting the alpha info to `CGImageAlphaInfo.first` to indicate the alpha channel as the first component in the pixel), and the rendering intent (employing the default intent).

```swift
let format = vImage_CGImageFormat(
    bitsPerComponent: 8,
    bitsPerPixel: 32,
    colorSpace: CGColorSpaceCreateDeviceRGB(),
    bitmapInfo: CGBitmapInfo(rawValue: CGImageAlphaInfo.first.rawValue),
    renderingIntent: .defaultIntent
)!
```

We then create a `vImage_Buffer` object to store the image data. Utilize the `cgImage:` initializer of the `vImage_Buffer` class, providing the input image and the image format we previously defined. We also use a `guard` statement to handle the case where the buffer creation fails.

```swift
let histogramSourceCGImage = image
guard var histogramSourceBuffer = try? vImage_Buffer(
    cgImage: histogramSourceCGImage,
    format: format
) else {
    return []
}

defer { histogramSourceBuffer.free() }
```

We use a `defer` statement to free the memory allocated for the `histogramSourceBuffer` after finishing the calculation. This is important to prevent memory leaks and ensure efficient use of system resources.
Note that we use a `var` for `histogramSourceBuffer` since we will modify it later in the function.

Following that, create four arrays to store the pixel counts for each color channel, initializing them with 256 elements each. Use the `withUnsafeMutableBufferPointer` method for each array to obtain a pointer to its underlying memory, which will then be provided to the `vImageHistogramCalculation_ARGB8888` function.

```swift
var histogramRed = [vImagePixelCount](repeating: 0, count: 256)
var histogramGreen = [vImagePixelCount](repeating: 0, count: 256)
var histogramBlue = [vImagePixelCount](repeating: 0, count: 256)
var histogramAlpha = [vImagePixelCount](repeating: 0, count: 256)

histogramRed.withUnsafeMutableBufferPointer { red in
    histogramGreen.withUnsafeMutableBufferPointer { green in
        histogramBlue.withUnsafeMutableBufferPointer { blue in
            histogramAlpha.withUnsafeMutableBufferPointer { alpha in
                // Proceed with the next step here...
            }
        }
    }
}
```

Next, create an array of pointers to these four arrays (note that the specified order is important), which we will pass to the `vImageHistogramCalculation_ARGB8888` function:

```swift
// Inside the previous chain...
// Order is important!
var histogramBins = [alpha.baseAddress, red.baseAddress, green.baseAddress, blue.baseAddress]

histogramBins.withUnsafeMutableBufferPointer { bins in
    let error = vImageHistogramCalculation_ARGB8888(
        &histogramSourceBuffer,
        bins.baseAddress!,
        vImage_Flags(kvImageNoFlags)
    )
    
    guard error == kvImageNoError else {
        print("Failed to calculate histogram!")
        return
    }
}
```

The `vImageHistogramCalculation_ARGB8888` function computes the histogram of an image in the ARGB8888 format, a prevalent pixel format used in iOS and macOS. This function requires three arguments: the input image buffer, the histogram bin array, and a set of image processing flags (here, we provide `kvImageNoFlags` to specify that we don't want to alter the input image in any manner).

Once the histogram for each color channel is calculated, save the pixel counts in a 2D array and return it from the function:

```swift
let channels = [histogramRed, histogramGreen, histogramBlue, histogramAlpha]
return channels
```

Great, the function is ready! Now you can call it for your `CGImage` and get the result. You can use this histogram to analyze and further process the image:

```swift
let histogram = calculateHistogram(image: cgImage)
print(histogram) // Prints [[35583, 5338, 3171, 1705, 1806, 1880, 1527, 906, 975, ...], [...], [...], [...]]
```

The complete code can be found in the [GitHub Gist](https://gist.github.com/igooor-bb/1a4c62670730e533f83be2dae85c78ee).
