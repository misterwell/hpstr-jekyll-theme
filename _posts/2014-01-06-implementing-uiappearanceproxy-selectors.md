---
title: Implementing UIAppearanceProxy Selectors
layout: post
tags: [iOS]
image:
  feature: abstract-10.jpg
---

I’ve been using the splendid [M13Checkbox](https://github.com/Marxon13/M13Checkbox) open-source library in one of my projects, and while updating the UI to support iOS 7’s flat motif I found that the library had implemented UIAppearance selectors, and there was much rejoicing:

```objc
//Appearance
@property (nonatomic, assign) BOOL flat UI_APPEARANCE_SELECTOR;
@property (nonatomic, assign) CGFloat strokeWidth UI_APPEARANCE_SELECTOR;
@property (nonatomic, retain) UIColor *strokeColor UI_APPEARANCE_SELECTOR;
@property (nonatomic, retain) UIColor *checkColor UI_APPEARANCE_SELECTOR;
@property (nonatomic, retain) UIColor *tintColor UI_APPEARANCE_SELECTOR;
@property (nonatomic, retain) UIColor *uncheckedColor UI_APPEARANCE_SELECTOR;
@property (nonatomic, assign) CGFloat radius UI_APPEARANCE_SELECTOR;
```

However, after using all of these in my code, I found that not only were some of the appearance methods not working, but one was causing a crash! Surely I was doing something wrong, but to understand exactly what, I dove into the M13Checkbox source and read up on implementing custom UIAppearance selectors. Turns out, I had to go a bit beyond that to understand how to fix it all!

As always, I started with the WWAD rule, or “What Would Apple Do?”. From the documentation, it sounds like creating appearance selectors is simple & straightforward:

To support appearance customization, a class must conform to the UIAppearanceContainer protocol and relevant accessor methods must be marked with `UI_APPEARANCE_SELECTOR`.

After reading up on the UIAppearanceContainer documentation, I didn’t come across any gotchas or important notes that mention what would cause a breakdown in functionality. I did find that BOOL wasn’t a valid data type for UIAppearance selectors, so by using NSInteger instead that crash went away. I then dove into the source code and found that M13Checkbox’s authors had done everything by the book (which essentially boils down to the source I included above), so I still couldn’t explain why things weren’t working. Specifically, the UIColor appearance selectors weren’t taking effect.

Thanks to a post by Peter Steinberger titled [UIApperance for Custom Views](http://petersteinberger.com/blog/2013/uiappearance-for-custom-views/), I was able to understand the how & why of UIAppearance selectors, and found that the root cause of the issue was a change that another developer on the team had made to try to fix something else (turns out that something else didn’t even need fixing). Peter sums it up in his post:

Lesson: Only use direct ivar access in the initializer for properties that comply to `UI_APPEARANCE_SELECTOR`.

I strongly urge anyone that is implementing a custom class with intentions of supporting appearance customization to read the full post by Peter… if anything it’s extremely interesting to see how Apple tries to do some of the magic that makes the iOS SDK run.
