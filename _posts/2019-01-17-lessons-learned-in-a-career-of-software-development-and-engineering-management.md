---
layout: post
title: Lessons Learned in a Career of Software Development and Engineering Management
description: "Join me as I try to unpack what 10+ years of software engineering has taught me."
---

Despite my best efforts to ignore it, I've been doing software development for a long, long time. And in that time, I like to think I've continued to hone my skills and be a better software engineer as well as a better person, but remain mindful that I can always improve and become better if I keep an open & curious mind.

I'll be the first to admit that software development isn't for everyone -- specifically, I'm of the mindset that in order to excel at writing code, you specifically need to be an *engineer*. This statement might intimidiate some people who feel like they don't have a formal education in engineering, but it shouldn't -- after all, take a look at the definition of an *engineer* according to whatever source Apple uses for it's dictionary application:

> engineer: noun. a person who designs, builds, or maintains engines, machines, or public works.

In my experience, this simply boils down to one thing: _**problem solving**_. Even the simplest coding tasks will require some form of problem solving where the costs & benefits of the choices in front of a developer must be weighed, and a decision must be made that will have a lasting effect on the code base. In my experience, this is primarily where bad developers faulter -- they just write code without considering the short- and long-term impacts of every line they're writing.

### Always take the time to refactor common logic rather than doing a copy/paste

It's tough to communicate the enormous value of doing this without experiencing the heartache of NOT doing it first hand, but I can't stress this point enough. When a snippet of code is copied and pasted elsewhere, it's often disasterous down the road for the key reason that the copied bit of code will some day be modified, and whether the other areas will get updated as well is anyone's guess. Behavior will be different depending on which version of the copied code is being executed. Extending the code in the future will be a pain and the pasted bits of code might not be remembered when the original is getting updated.

The most common challenge I've run into is I'm under a time crunch and I'm faced with a code block that could be used in two places but their needs differ in some key ways. But the best thing to do is to take the common code bit and take the time to turn it into a function with parameters that can be used to achieve the behavior needed by both places. This can take a bit of additional time, but the more you do it the better you'll get at it, and the time savings you and your team will experience later on can't be understated.

### Don't be afraid to Google problems (both simple & tough)

Engineers are, stereotypically, a prideful bunch. As a species, we typically don't like to ask for help, and there's a notion that if you use Google to find an answer you're not a good programmer. That's bollix, I say.

First off, unless you have an idetic or photographic memory, it's probably impossible to commit to memory the entirety of knowledge that is needed to handle the typical problem set that developers are up against. As an iOS developer, I know many iOS frameworks extensively, but UIKit has hundreds, fi not thousands, of classes, structs, and typedefs -- and on top of that there are often nuances of how each of them can and should be used.

Secondly, the Internet has a wealth of information, but conversely it also has an enormous amount of incorrect and outdated information as well. Not only is  effectively creating a concise search term to reach relevant content a skill in itself, but efficiently sifting through the results to actually find relevant and useful content takes skill as well. When I was responsible for merging two complex, open-source C++ libraries together I was constantly searching for information and instructions to help lead me to find information that would help me cobble together a coherent combined library along with a functional GNUMake build system. Had I sat there and tried to figure it out myself, or only use the resources & experience our company had, it would've never been completed.

What is key is to integrate the content you find on the internet into your experience base, rather than just taking it and pasting into what you're working on. By doing the former, you have it in your experience toolbelt to come back to and use later (even if you have to look it up again), but doing the latter leaves you just about as experienced as you were before you found the answer.

### Avoid reinventing the wheel

There are many engineers out there who believe that each dependency you bring into your code base represents technical debt that exposes you to unnecessary risk, and that as a general rule engineers should strive to try to build as much as possible first-hand as it ensures the team will have complete control and responsiblity over what's contained in the code base. I can't say that I disagree with this, but I think it's a shortsighted approach that ultimately is bad for the _business_ of development. I'll elaborate with an example:

At Hathway, our developers decided to create a library to specifically handle the networking and JSON serialization/deserialization of data for a couple of reasons, one of them being that while the popular Alamofire library already did this, it was large and might bring some baggage and bloat to our application that we were trying to keep lean & mean. So we created a fairly simple library to do the minimum needed for our app. This module we craeted has certainly served us well, but over the years there have been some key issues that have impacted us:

1. __Despite the module's simplicity, there were bugs that wouldn't have existed within a more mature third-party library.__

    Some were found quickly and addressed, but others were latent and were the root cause of gremlins in all of our apps that used the module -- one specific case had to do with HTTP caching, which I personally spent numerous hours tracking down and addressing.

2. __Over time we had to modify the library's architecture to support more advanced functionality that the third-party library already had within it.__

    I definitely agree with the decision to keep our library simple to begin with -- after all, complexity equals cost -- but as we built more advanced things our library reached its breaking point.

3. __Once we reached a point where we needed to switch to a more robust third-party library, the cost to change was high.__

    By the time that breaking point was hit, we had already made compromises in our apps and the library to workaround the deficiencies of our library, so moving to another one wasn't as simple as we'd hoped. Obviously any switch of this type will require _some_ refactoring to adjust, but our feeling was it seemed like more effort than what a switch between two mature libraries would have been.

In the end, my analysis of this is that in order to get some small gains in compile & app execution time, we likely spent a disproportionate amount of time managing & maintaining our own library instead of just using Alamofire.

With all that being said, it's critical that when you include a dependency in your code base, you make damn sure that it's:

1. Got a good number of contributors
2. Is up to date with its platform (i.e. iOS/Android)
2. Is widely used by the community
3. Is accepting to outside issues & contributions
4. Doesn't have an alarming number of real issues logged

### Plan, but don't overplan

This one might not apply to all engineers as everyone has a different way of thinking, but personally I can't effectively plan how to build something without building a little bit of it and then taking a step back to analyze and plan where to go from there. It's kind of a 1 step forward, 2 steps back, 5 steps forward kind of thing.

Be agile in your architecture planning -- try to deliver small bits of fully functional code and then build on top of that, and if you realize one of your completed bits needs to be changed to work better, take the time to do it. Don't be afraid to make mistakes, but don't ignore or workaorund your mistakes -- fix them.

### Share your work in progress early and be vigialant about asking for feedback

Again, engineers are often prideful and, in my experience, don't want to show off their work until it's done & amazing. But what if you build the wrong thing? One of my sayings is that "in software development, there's usually hundreds of ways to build something, and less than 5 of them are ever actually right." There are so many ways to build something wrong, and showing work in progress code to your peers gives you another set of eyes to provide a sanity check that you're heading in the right direction, rather than waiting until the end for someone to tell you that you spent days getting to the wrong place.

This is especially true for tech leads or engineering managers -- sure, you want to trust that your reports are doing everything perfectly from the get-go... but if that were the case there wouldn't be tech leads or engineering managers. The sooner an issue is identified the cheaper it will be to fix, so get invovled early and do a peer programming review to make sure your people are on the right track.

### If you can't diagram it, you can't build it

I suck at drawing. Truly. But one of the things I love doing is creating engineering diagrams -- system architectures, sequence diagrams, UML diagrams, and so many more. Drawing out what you intend to build is the most effective way to do these key things:

1. Communicate a plan and make sure everyone is on the same page
2. Force you to do a requisite amount of planning to help you figure out how you're going to build something
3. Provide an artifact to refer back to for new engineers coming onto the project, the team to discuss around, and so much more

And it doesn't have to be fancy -- I use OmniGraffle to create some really amazing diagrams but whiteboarding is a great way to rapidly diagram something and sending a picture of it out to the team is just about as effective (assuming your handwriting isn't complete chicken scratch!).

### If you can't explain how you're going to refactor something, you're not ready to refactor

Over the years, I've had engineers reporting to me that come to me and can very clearly articulate why a library or module is constraining or blocking them, and that will lead them to the next logic step of recommending a refactor to fix the deep-seeded issues. Sometimes, your team will have the bandwidth to do a large & beneficial refactor, so you may be inclined to set your engineers loose and trust that they'll refactor it to be much better. But this is often not happens -- ultimately the goal of a refactor is to make a component better than it was before, but sometimes engineers get ancy and just rebuild the component without truly solving all the problems at hand.

So the first question I ask when presented with an engineer asking to refactor a component is "how exactly are you going to fix the problems you've cited?". It seems like a simple question, but you should demand a high level of detail in the response. Sometimes the answer will be "we just need to abstract XYZ", to which I would respond "but what problems will that solve?". Never build a solution in search of a problem -- start with the problem and articulate a solution that will solve it.

Engineers often like to chase shiny things and build cool things with new technologies -- make sure a refactor is focused on solving identified problems rather than just doing something cool.

## One Final Note

If there's one thing I've learned in all of the years I've been writing code and managing people, it's that there's always more to learn and ways to improve what you're doing, but those things take time to happen. A lot of people think that engineering is solely a left-brain activity, but I truly believe that the best engineers use both sides of their brain to build amazing things, so to be the best engineer you can be you have to work on improving both sides of your brain over the course of your career.

And, in case it needed to be said, I've got a long way to go too. If there's one thing I'm certain of, it's that I've got as much to learn in the next 10 years as I have in the past 10.
