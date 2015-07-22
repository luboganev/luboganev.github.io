---
layout: post
title: Android clean architecture part 2
description: "This post discusses articles and blog posts I have used as information sources, while trying to come up with some rules and patterns of an Android architecture"
modified: 2015-07-23
category: drafts
tags: [android, tip, pattern, architecture, software development]
share: true
---

### The drama we have avoided
I guess every developer has at some point tried to find some clean architecture model. Most of the time, however, this happens in a pretty late stage of the project, thus incorporating some particular architecture more often than not requires major refactoring and complete rebuild of some components. This is a pretty high cost, which the product management often does not want to pay, because it will probably not bring any benefit or improvement felt by the product's users and the project's stakeholders. Therefore it never happens and the problem keeps deepening until somebody burns out, screams loudly or becomes increasingly unhappy with his job and eventually quits. In order to avoid such dramatic events, our team has looked for some clean architecture from the very beginning of the implementation of our iOS app.

### Uncle Bob's clean architecture
We went a long way back to the software engineering theory and we discovered Uncle Bob's Clean architecture.

![Uncle Bob's clean architecture]({{ site.url }}/images/2015-07-23-clean-architecture-pt2/CleanArchitecture.jpg)

The original blog post describing in detail this architecture can be found here: [Link](http://blog.8thlight.com/uncle-bob/2012/08/13/the-clean-architecture.html). Although it dates back to 2012, it describes a fundamental approach, which could be used when building flexible and testable software in general. The most important aspect of this achitecture is the *Dependency rule*. 

> Nothing in an inner circle can know anything at all about something in an outer circle. In particular, the name of something declared in an outer circle must not be mentioned by the code in the an inner circle. That includes, functions, classes. variables, or any other named software entity.

In addition, all connections and communication between circles should be abstractly implemented, e.g. by using simple POJOs and Interfaces when building for Android in Java, or using Protocols and standart classes, when building in Objective-C and Swift in iOS. Such an architecture is extremely flexible and testable. Practically, the implementation of every single component from a particular circle could be completely changed or mocked, without having to implement any changes in other parts of the code. 

This all looks pretty nice, but is also very vague and abstract. How would one go on with implementing it in a mobile app? The answer is - it has already been updated for the context of mobile apps by some smart people and is called VIPER architecture for iOS.

### V.I.P.E.R architecture for iOS

![Anatomy of a VIPER]({{ site.url }}/images/2015-07-23-clean-architecture-pt2/viper_diagram.jpg)

The original blog post describing in detail this architecture can be found here: [Link](http://mutualmobile.github.io/blog/2013/12/04/viper-introduction/). Further high quality read is the objc.io article on this topic which can be found here: [Link](http://www.objc.io/issues/13-architecture/viper/).


It started as an experiment, with the agreement that we would go away from it in case it proves problematic. But it didn't. 