---
layout: post
title: Using Vector Graphics on iOS
description: "How to make generated Paintcode vector graphics more robust on iOS"
tags: [iOS]
image:
  feature: post-vectors-on-ios/vectorHeader.png
---

In the past few projects I’ve been a part of, I’ve been using tools like [Qwarkee](https://itunes.apple.com/us/app/qwarkee/id498340809?mt=12) and [PaintCode](https://itunes.apple.com/us/app/paintcode/id507897570?mt=12) to create Obj-c Quartz drawing code, and have been impressed with the results. I’ve found that it wasn’t inherently obvious how the generated code can be integrated in a simple & modular way. I recently created the [MMScalableVectorView repository](https://github.com/misterwell/MMScalableVectorView) on GitHub that attempts to address this observed deficiency. Issues/Pulls encouraged. For the interested, read on for my thoughts & conclusions that led me to this.

## Vectors & Bitmaps: Same End, Different Means

Not everyone is familiar with vector images and how they’re different from traditional bitmap images. I am certainly not a graphics compression & performance expert, but I do have enough experience to help others understand how the two are different. Bitmap images are exactly what they sound like – a map of bits that come together to form an image. Each bit defines its color value, and whatever is reading the bitmap draws them in sequence to form the image. The most common bitmap used on the internet is PNG, but GIF and JPEG are also very popular as well, so anyone who has used the internet has come into contact with a bitmap image. Vector images attempt to reach the same end result as bitmaps, but they go about it in a completely different way. Rather than defining a map of bits, vector images record the steps necessary to actually draw the content contained in an image. These instructions can be extremely simple, like “draw a 2 pixel wide black line from point A to point B”, or they can be complicated, such as “draw a bezier curve using this set of control points and then fill the path I’ve been drawing with a gradient with these other control points”. By following these instructions, the computer actually re-draws the image itself when presenting the content to the user.

## Why Does It Matter?

Why should the end-user care whether they’re being shown bitmap images or vector images? Simply put, they shouldn’t. But more precisely, what matters is whether the UI being presented looks good & functions well in whatever environment it is accessed from. For mobile applications and responsive web sites, that means adapting to the user’s screen size without sacrificing graphical fidelity. And that’s where these two paths diverge pretty significantly. As we’ve learned, bitmaps are essentially just a picture of an image, while vectors are a set of instructions on exactly how to recreate that image. So what happens when we ask these images to fit a space thats larger or smaller than what it was originally intended to fill? For bitmaps, the results should be familiar to just about anyone:

<figure>
    <img src="{{ site.url }}/images/post-vectors-on-ios/bitmapExample.png" alt="">
    <figcaption>Bitmap image blown up by a factor of 2</figcaption>
</figure>
{: .image-center}

The original size looks great, but blowing the image up by only 2x makes it look blurry. Why is this? Simply put, there’s not enough information in the original bitmap representation of the image to recreate it at a larger size. It would be like someone handing you a 4×6 photo and asking you to create an 8×10 without the negative. So how do vector images look when they’re blown up? It can’t be that much better, can it?

<figure>
    <img src="{{ site.url }}/images/post-vectors-on-ios/vectorExample.png" alt="">
    <figcaption>Vector image blown up by a factor of 2</figcaption>
</figure>
{: .image-center}

The results are dramatically different! This is because the set of instructions that describe how to create the image is more powerful than just having the image itself. By doubling or tripling the distances and control points that a vector graphic describes, a computer can perfectly recreate that image at a larger size than the graphic was originally intended to fill. Beyond this, vectors have other advantages and disadvantages worth considering:

### Pros

*   Images can be scaled up & down without degradation of quality
*   Only one version of the vector image needs to be created to suit all sizes.
*   No need to create separate image files for retina & non-retina displays.
*   On many platforms, vector graphics are drawn using the system’s GPU, which can yield enormous performance gains, especially when more and more graphics are being drawn on the screen at the same time.

### Cons

*   Vector graphics are not ubiquitously understood by developers (hence this article)
*   Not natively supported by iOS or Android platforms, and the middleware that converts vector images into native iOS & Android drawing code isn’t always reliable or free.
*   Gains are diminished if the UI design is pretty static or images aren’t reused throughout the application in various sizes.
*   Only an option if the graphics were created in a vectorized format — converting bitmaps into vectors is not a good idea.

## Why & How I Use Vectors

As I’ve spent time on iOS apps over the years, I found myself using vector graphics more and more for the following reasons:

1.  The applications’ UI was never finalized enough to be confident that the bitmaps being created wouldn’t have to be re-created down the line if (and, consequently, when) the UI changed.
2.  Certain icons were being used across the app in various sizes, and because of point #1 above, managing the various bitmap images was going to be a nightmare.
3.  Some of the table view cells in the applications could display up to 4 of these images at a time, so the performance gains of vector images would help contribute to keeping things smooth, especially during scrolling.

So I decided to embark upon a journey to use vectors. I did some digging around the internet, and found 2 tools that specialize in converting SVG files into native iOS Objective-C drawing code. The first one I tried was the free option, called [Qwarkee Light](http://itunes.apple.com/us/app/qwarkee/id498340809?ls=1&mt=12 "Qwarkee Lite") (which has a paid counterpart, [Qwarkee](https://itunes.apple.com/us/app/qwarkee/id498340809?mt=12)). This app is a straightforward SVG to Objective-C conversion tool, so it doesn’t have a built-in editor that allows editing the vector you’re converting. But assuming you’ve got another application to create & edit vector images, such as [Sketch](https://itunes.apple.com/us/app/sketch/id402476602?mt=12) or Adobe Illustrator, this option would probably be fine. If you want an application that allows editing vectors as well as exporting them, then spending $100 for [PaintCode](https://itunes.apple.com/us/app/paintcode/id507897570?mt=12) seems to be money very well spent. I won’t go into more detail on the applications and their features, because all that really matters is that both will spit out vector drawing code that looks something like this:

```objc
//// General Declarations
CGContextRef context = UIGraphicsGetCurrentContext();

//// Color Declarations
UIColor* circleFillColor = [UIColor colorWithRed: 0.247 green: 0.431 blue: 0.71 alpha: 1];
UIColor* white = [UIColor colorWithRed: 1 green: 1 blue: 1 alpha: 1];

//// Abstracted Attributes
NSString* textContent = @"Test Text!";

//// User circle
{
    //// User Portrait
    {
        CGContextSaveGState(context);

        //// Clip Bezier 2
        UIBezierPath* bezier2Path = [UIBezierPath bezierPath];
        [bezier2Path moveToPoint: CGPointMake(336.11, 62.88)];
        [bezier2Path addCurveToPoint: CGPointMake(336.11, 336.12) controlPoint1: CGPointMake(411.56, 138.33) controlPoint2: CGPointMake(411.56, 260.67)];
        [bezier2Path addCurveToPoint: CGPointMake(62.89, 336.12) controlPoint1: CGPointMake(260.66, 411.57) controlPoint2: CGPointMake(138.34, 411.57)];
        [bezier2Path addCurveToPoint: CGPointMake(62.89, 62.88) controlPoint1: CGPointMake(-12.56, 260.67) controlPoint2: CGPointMake(-12.56, 138.33)];
        [bezier2Path addCurveToPoint: CGPointMake(336.11, 62.88) controlPoint1: CGPointMake(138.34, -12.57) controlPoint2: CGPointMake(260.66, -12.57)];
        [bezier2Path closePath];
        [bezier2Path addClip];

        //// Circle fill Drawing
        UIBezierPath* circleFillPath = [UIBezierPath bezierPath];
        [circleFillPath moveToPoint: CGPointMake(336.11, 62.26)];
        [circleFillPath addCurveToPoint: CGPointMake(336.11, 335.5) controlPoint1: CGPointMake(411.56, 137.72) controlPoint2: CGPointMake(411.56, 260.05)];
        [circleFillPath addCurveToPoint: CGPointMake(62.89, 335.5) controlPoint1: CGPointMake(260.66, 410.96) controlPoint2: CGPointMake(138.34, 410.96)];
        [circleFillPath addCurveToPoint: CGPointMake(62.89, 62.26) controlPoint1: CGPointMake(-12.56, 260.05) controlPoint2: CGPointMake(-12.56, 137.72)];
        [circleFillPath addCurveToPoint: CGPointMake(336.11, 62.26) controlPoint1: CGPointMake(138.34, -13.19) controlPoint2: CGPointMake(260.66, -13.19)];
        [circleFillPath closePath];
        circleFillPath.miterLimit = 4;

        [circleFillColor setFill];
        [circleFillPath fill];

        //// Bezier 3 Drawing
        UIBezierPath* bezier3Path = [UIBezierPath bezierPath];
        [bezier3Path moveToPoint: CGPointMake(293.36, 315.67)];
        [bezier3Path addCurveToPoint: CGPointMake(235.97, 257.83) controlPoint1: CGPointMake(249.87, 299.82) controlPoint2: CGPointMake(235.97, 286.46)];
        [bezier3Path addCurveToPoint: CGPointMake(255.07, 214.79) controlPoint1: CGPointMake(235.97, 240.64) controlPoint2: CGPointMake(249.25, 246.25)];
        [bezier3Path addCurveToPoint: CGPointMake(271.47, 184.77) controlPoint1: CGPointMake(257.49, 201.73) controlPoint2: CGPointMake(269.22, 214.58)];
        [bezier3Path addCurveToPoint: CGPointMake(265.06, 169.94) controlPoint1: CGPointMake(271.47, 172.89) controlPoint2: CGPointMake(265.06, 169.94)];
        [bezier3Path addCurveToPoint: CGPointMake(269.59, 138.83) controlPoint1: CGPointMake(265.06, 169.94) controlPoint2: CGPointMake(268.31, 152.36)];
        [bezier3Path addCurveToPoint: CGPointMake(234.97, 74.51) controlPoint1: CGPointMake(270.79, 125.97) controlPoint2: CGPointMake(275.63, 79.36)];
        [bezier3Path addCurveToPoint: CGPointMake(199.5, 61.96) controlPoint1: CGPointMake(232.65, 64.21) controlPoint2: CGPointMake(213.8, 61.96)];
        [bezier3Path addCurveToPoint: CGPointMake(140.56, 84.88) controlPoint1: CGPointMake(170.77, 61.96) controlPoint2: CGPointMake(150.96, 72.72)];
        [bezier3Path addCurveToPoint: CGPointMake(129.4, 138.83) controlPoint1: CGPointMake(129.1, 98.26) controlPoint2: CGPointMake(128.57, 130)];
        [bezier3Path addCurveToPoint: CGPointMake(133.92, 169.94) controlPoint1: CGPointMake(130.67, 152.36) controlPoint2: CGPointMake(133.92, 169.94)];
        [bezier3Path addCurveToPoint: CGPointMake(127.52, 184.77) controlPoint1: CGPointMake(133.92, 169.94) controlPoint2: CGPointMake(127.52, 172.89)];
        [bezier3Path addCurveToPoint: CGPointMake(143.91, 214.79) controlPoint1: CGPointMake(129.76, 214.58) controlPoint2: CGPointMake(141.49, 201.73)];
        [bezier3Path addCurveToPoint: CGPointMake(163.01, 257.83) controlPoint1: CGPointMake(149.75, 246.26) controlPoint2: CGPointMake(163.01, 240.64)];
        [bezier3Path addCurveToPoint: CGPointMake(105.62, 315.67) controlPoint1: CGPointMake(163.01, 286.46) controlPoint2: CGPointMake(149.12, 299.82)];
        [bezier3Path addCurveToPoint: CGPointMake(33.65, 358.84) controlPoint1: CGPointMake(62, 331.58) controlPoint2: CGPointMake(33.65, 347.79)];
        [bezier3Path addCurveToPoint: CGPointMake(33.65, 395.98) controlPoint1: CGPointMake(33.65, 369.89) controlPoint2: CGPointMake(33.65, 395.98)];
        [bezier3Path addLineToPoint: CGPointMake(199.5, 395.98)];
        [bezier3Path addLineToPoint: CGPointMake(365.33, 395.98)];
        [bezier3Path addCurveToPoint: CGPointMake(365.33, 358.84) controlPoint1: CGPointMake(365.33, 395.98) controlPoint2: CGPointMake(365.33, 369.89)];
        [bezier3Path addCurveToPoint: CGPointMake(293.36, 315.67) controlPoint1: CGPointMake(365.33, 347.79) controlPoint2: CGPointMake(336.99, 331.58)];
        [bezier3Path closePath];
        bezier3Path.miterLimit = 4;

        [white setFill];
        [bezier3Path fill];

        CGContextRestoreGState(context);
    }

    //// Bezier 4 Drawing
    UIBezierPath* bezier4Path = [UIBezierPath bezierPath];
    [bezier4Path moveToPoint: CGPointMake(336.11, 62.88)];
    [bezier4Path addCurveToPoint: CGPointMake(336.11, 336.12) controlPoint1: CGPointMake(411.56, 138.33) controlPoint2: CGPointMake(411.56, 260.67)];
    [bezier4Path addCurveToPoint: CGPointMake(62.89, 336.12) controlPoint1: CGPointMake(260.66, 411.57) controlPoint2: CGPointMake(138.34, 411.57)];
    [bezier4Path addCurveToPoint: CGPointMake(62.89, 62.88) controlPoint1: CGPointMake(-12.56, 260.67) controlPoint2: CGPointMake(-12.56, 138.33)];
    [bezier4Path addCurveToPoint: CGPointMake(336.11, 62.88) controlPoint1: CGPointMake(138.34, -12.57) controlPoint2: CGPointMake(260.66, -12.57)];
    [bezier4Path closePath];
    [white setStroke];
    bezier4Path.lineWidth = 1;
    [bezier4Path stroke];
}
```

With this code in hand (or, more specifically, your clipboard), all you have to do is create a new UIView subclass, and paste it into its drawRect method. That’s it! So let’s give it a try with various view sizes and watch the power of vectors take over!

<figure>
    <img src="{{ site.url }}/images/post-vectors-on-ios/normalVector.png" alt="">
    <figcaption>Whiskey Tango Foxtrot??!?</figcaption>
</figure>
{: .image-center}

Surely we’ve done something wrong — we’ve used vector images, yet they’re not scaling, and this seems to defeat the whole purpose of using vector graphics! Let’s review the first few lines of the drawing code to see if we can pinpoint the problem:

```objc
//// Clip Bezier 2
UIBezierPath* bezier2Path = [UIBezierPath bezierPath];
[bezier2Path moveToPoint: CGPointMake(336.11, 62.88)];
[bezier2Path addCurveToPoint: CGPointMake(336.11, 336.12) controlPoint1: CGPointMake(411.56, 138.33) controlPoint2: CGPointMake(411.56, 260.67)];
[bezier2Path addCurveToPoint: CGPointMake(62.89, 336.12) controlPoint1: CGPointMake(260.66, 411.57) controlPoint2: CGPointMake(138.34, 411.57)];
[bezier2Path addCurveToPoint: CGPointMake(62.89, 62.88) controlPoint1: CGPointMake(-12.56, 260.67) controlPoint2: CGPointMake(-12.56, 138.33)];
[bezier2Path addCurveToPoint: CGPointMake(336.11, 62.88) controlPoint1: CGPointMake(138.34, -12.57) controlPoint2: CGPointMake(260.66, -12.57)];
[bezier2Path closePath];
[bezier2Path addClip];
```

Notice that all of the lines marking points and drawing curves use static coordinate values, which means that there are no adjustments made to the coordinates based on the frame the vector is being drawn in. And according to [Apple’s documentation](https://developer.apple.com/library/ios/documentation/graphicsimaging/conceptual/drawingwithquartz2d/dq_affine/dq_affine.html#//apple_ref/doc/uid/TP30001066-CH204-CJBBHBEF) (emphasis added):

You manipulate the CTM to rotate, scale, or translate the page before drawing an image, thereby transforming the object you are about to draw.

So unless we apply a transform before drawing our image, we’re out of luck with regards to actually drawing our vector at a different scale. So what about using the transform property of UIView on our vector subclass? [Apple’s documentation](https://developer.apple.com/library/IOs/documentation/WindowsViews/Conceptual/ViewPG_iPhoneOS/CreatingViews/CreatingViews.html#//apple_ref/doc/uid/TP40009503-CH5-SW4):

View transforms alter the final rendered appearance of the view

Note: I’m not 100% convinced that this means the transform property is always applied to the rendered bitmap of a UIView, but doing some tests on Apple’s own UIKit controls seem to indicate this is the case. Also, doing this would require that the superview or parent controller know details about the underlying implementation of the UIView subclass, which goes against encapsulation and view hierarchy practices. At a minimum, the parent of the vector subclass would need to know the size of the original vector, and then perform transformations before instructing the subclass to draw itself, and that doesn’t sound like an easy solution. So the ideal solution to make static vector graphics code copied from Qwarkee or PaintCode would provide the following:

1.  The ability to use the code generated by Qwarkee or PaintCode without modification. This ensures that code changes made to make the vector code scalable don’t need to be re-applied anytime the graphic changes.
2.  Support scaling, ideally by using UIView’s existing contentMode property If the vector graphic is going to be a UIView subclass, it should play by UIView’s rules. Also, this means we don’t need to create a new property to define the desired vector scaling behavior.
3.  Maintain performance gains of using vector graphics Regardless of the solution, we obviously still want vector graphics to scale up and look beautiful, as well as provide the CPU performance gains achieved by drawing with the Quartz/CoreGraphics framework.

## Enter MMScalableVectorView

I decided to solve these problems so that I could more easily use vector graphics in my iOS applications, and the result is [MMScalableVectorView](https://github.com/misterwell/MMScalableVectorView). It’s an extremely simple solution and it’s not a magic bullet, but at a minimum it provides the ability to use vector images as UIView objects that honor the view’s contentMode property when the vector is drawn. Observe the results:

<figure>
    <img src="{{ site.url }}/images/post-vectors-on-ios/scaledVector.png" alt="">
    <figcaption>That's more like it.</figcaption>
</figure>
{: .image-center}

### Instructions

1.  Create a new class to represent the graphic, and make sure it inherits from MMScalableVectorView.
2.  Implement the following methods:
    1.  Paste the Qwarkee/PaintCode drawing code into drawInCurrentContext
    2.  Implement originalSize to return a CGSize structure for the native size of the vector graphic.
3.  Use the newly created class as a normal UIView, setting the contentMode property as desired before the view is drawn. This can be done in Interface Builder or programatically.

### Future Improvements

1.  UIView category with a block-based initializer for one-stop creation of vector-backed UIViews.
2.  Improving drawing when UIView’s transform property is used (?)
3.  Unit tests!

Please feel free to open issues or issue pull requests if you feel you can contribute to the project! And thanks for taking the time to read.