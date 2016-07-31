---
layout: post
title: Creating Google's Material Design Spinner on iOS
description: "Implementing the Google Material Design Spinner for iOS"
tags: [iOS]
image:
  feature: material-spinner-header.gif
---

TLDR; if you want Google’s Indeterminate Material Design spinner on iOS, check out my [MMMaterialDesignSpinner](https://github.com/misterwell/MMMaterialDesignSpinner) open-source library.

While I was working on the redesign of thetvdb, I plopped a UIActivityIndicatorView into a view and realized that the control looks incredibly out of date:

<figure>
    <img src="http://mikethinkingoutloud.com/wp-content/uploads/2014/06/ActivityIndicator.gif" alt="">
    <figcaption>iOS' built-in UIActivityIndicatorView in action.</figcaption>
</figure>
{: .image-center}


Meanwhile, Google has released their [Material Design UI](http://www.google.com/design/spec/material-design/introduction.html) update and their new activity indicator looks much more modern. Unfortunately, no one had gotten around to implementing it on iOS, so I found myself at [Kreuzberg](http://blog.kreuzbergcalifornia.com) one morning during the holiday break, and decided to knock out an open-source library for it.

## The Animation

The first step was to figure out exactly what was going on in the activity spinner, because even though I like the flow of it, it’s a bit disorienting when you actually try to follow what’s going on in it. Give it a try yourself:

<p data-height="240" data-theme-id="0" data-slug-hash="vEBpoL" data-default-tab="result" data-user="jczimm" data-embed-version="2" class="codepen">See the Pen <a href="http://codepen.io/jczimm/pen/vEBpoL/">Modern Google Loader in Pure CSS</a> by jczimm (<a href="http://codepen.io/jczimm">@jczimm</a>) on <a href="http://codepen.io">CodePen</a>.</p>
<script async src="//assets.codepen.io/assets/embed/ei.js"></script>  

See the Pen [Modern Google Loader in Pure CSS](http://codepen.io/jczimm/pen/vEBpoL/) by jczimm ([@jczimm](http://codepen.io/jczimm)) on [CodePen](http://codepen.io)

This example was done in HTML & CSS, so I was able to play with the animations and single each one out to understand what was going on. There are actually 2 discrete animations going on:

1.  The entire control is rotating at a constant speed, i.e. 1 revolution per 4 seconds.
2.  The beginning and end of the circle’s stroke path are changing, such that they both start at 0%, advance to new values, and then end at 100% (which, on a circle, is also 0%)

After breaking down the individual animations I realized it wasn’t very complicated, and moved on to implementation.

## The Math

After thinking about the two animations for a bit, I broke down what I’d actually be animating in the code.

The first animation, which I’ll call “the rotation”, is easy — the only variable is how fast the rotation occurs.

The second animation was a bit trickier. Since I wanted to make the control as lightweight & efficient as possible, I wanted to use CoreGraphics rather than using `drawRect:` or static images (a la an animated GIF). So I dug into the animatable properties of the `CAShapeLayer` class, and found that path property is not inherently animatable — bummer. That would’ve allowed me to simply specify a new `CGPath` arc using a `UIBezierPath` object and that would’ve been that.

Instead, I found that the `strokeStart` and `strokeEnd` properties are animatable, and that effectively would have the same effect as modifying the shape’s path. By modifying the path, I’d actually be modifying the underlying shape to represent arc pieces of a whole circle, but by modifying the stroke start and end, the path is always a complete circle and I’ll merely be specifying what part of it I want stroked.

To be clear, the best way to understand how the strokeStart and strokeEnd properties work is to imagine your shape is made from a single piece of string, cut it where the shape was first defined, and stretch it out into a straight line. 0 now represents the start of the straight line, and 1.0 (or 100%) represents the end of the straight line. So if `strokeStart` is `0.25` and `strokeEnd` is `0.75`, then the first and last quarter length of the string would not be stroked, but the middle would.

With that terminology, our animation #2 is simply the following, assuming our animation is 2 seconds long:

1.  `strokeStart` and `strokeEnd` both begin with a value of `0.0`.
2.  Animate `strokeStart` to `0.25` and strokeEnd to `1.0` over `1` second
3.  Keeping the animated values, animate `strokeStart` to `1.0` to meet with `strokeEnd`
4.  Reset `strokeStart` and `strokeEnd` to their original values of `0.0`.

With that in mind, let’s figure out how to code it.

## The Implementation

As I said, the animations are fairly simple, but CoreAnimation has some gotchas that always seem to come back to bite you even when you feel like you know what you’re doing. I won’t go into the nitty-gritty details of implementation (that’s why the code is open-source), but the animations I mentioned above ended up carrying over into code very well — #1 is a simple rotation transformation, and #2 is an animation group containing 3 animations that modify the stroke beginning and ending. Easy stuff.

So if you’ve been looking for a touch of Material Design on iOS, check out my [MMMaterialDesignSpinner](https://github.com/misterwell/MMMaterialDesignSpinner) open-source library. As always, issues & PR’s are encouraged.
