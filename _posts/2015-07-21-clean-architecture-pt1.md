---
layout: post
title: Android clean architecture part 1
description: "This is the first of a series of posts providing insights and know-how about a particular architecture style used while implementing a fresh production ready android application."
modified: 2015-07-21
category: blog
tags: [android, tip, pattern, architecture, software development]
share: true
---

### Y U NO POST
Lately I have been pretty busy visiting different meetups, having some proper social life, working on my private projects and building apps at my work, thus this huge delay since the last post I have written. I want to talk about something I am particularly proud of. At the beginning of the year I have done a whole week of experiments and have put a lot of thought in order to come up with an Android version of the architecture, which our team used while building our iOS application. The main goal was to use the same familiar "architectural language", which we have developped in our iOS application. This way we do not have to pay too high cost when switching between developing for iOS and Android. We have tried to establish a common language understood by all mobile developers, and I believe that we have succeeded. We have managed to build a pretty complex Android application with a lots of moving parts, which still feels quite right and does not bring us a lot of headaches. I have noticed that we tend to write almost the same subtasks for both platforms when implementing new features into our apps. I consider this a vital sign, showing that we have managed to achieve a harmony between our native implementations for iOS and Android. Therefore, I have decided to share some of the know-how and experience, which we have collected throughout the last months.

### Why to invest into architecture
I would like to start by talking why I think that apps architecture is an important topic to discuss and why it is worth to invest time and resources into building one. I guess all experienced Android engineers know how complex building and Android application can be. Well guess what? Building and iOS app is also pretty complex and developing for the iOS platform has its own major pains. One cannot just start buidling using a naíve approach, because sooner or later one will end up with a lot of god objects, no flexibility and dependencies flying everywhere. To sum up, at some point it will become a total mayhem and adding new functionality will be either extremely hard and needing major refactoring or completely impossible. Let us not underestimate the complexity of an app. Let's not even call it app, let's talk about a project or a system, because let's face it, a mobile application with a lot of functionality can be seen as complex system with front-end, back-end and storage.

It is sometimes pretty easy to start quickly hacking together a working application, but this approach cannot scale. It does not work well, when you first build a minimum viable product and then incrementally add a lot of extra features to it. It does not work well with product development in the agile software development world. And please do not tell me that planning a basic architecture is not an agile approach, because if you completely ignore this step, pretty soon you won't be able to be agile at all.

### Let's face it - building apps is complex
![Anatomy of an Android]({{ site.url }}/images/2015-07-21-clean-architecture-pt1/android_anatomy.jpg)
Let's look at some common Android framework stuff that one has to implement, work around or just manage when building applications. It probably does not have anything to do in particular with the main business logic of the applications, but one has to do it in order to have a working Android app at all.

* backwards compatibility
* configuration changes (orientation), UI state and threading
* local storage, data synchronisation with remote API
* fragile environment (spotty internet connection, and other apis)
* efficient usage of restricted resources (memory, bandwidth, cpu) 
* beautiful UI with proper Material design and good performance

Managing the above stuff is not trivial, but you probably have to do all of it in your app as well. The SDK itself is also not really trivial so let's stop underestimating it. There are so many components, patterns and functinality, some of which work only with particular other componets. Just think about the following list of components and patterns you probably use on daily basis:

* Activities, Services, Broadcast Receivers, Content Providers, Bundles, Intents
* Fragments, Views, Notifications, Resources
* Databases, IPC, Threads, Storage
* Thousands of APIs related to services and specific hardware features of devices
* Support libraries and external third party libraries

And this whole complexity is there, even before you start implementing the stuff that your app should do. So the question is, where to add the code which defines what your application does. Should you put it inside and Activity maybe? Let's see what is a common functionality, which an activity code will manage:

* lifecycle, state save and restore (Bundles, Parcelables)
* multiple entry points (Intents, Share, Notifications deep links)
* non-trivial navigation (back, up, backstack, processes, Intent flags)
* Views and Fragments inside and their communication (interfaces, callbacks, fragment transactions, inflation)
* transitions, shared hero elements, alternative layouts and resources for devices, orientation etc.

That's a lot of code already there, which handles a lot of stuff still having nothing to do you with your code business logic. Programming the whole business logic there will make the code really unreadable, and you will probably not be able to keep track of what is going on, due to the huge bloated class. We need a better separation of concerns and a set of common patterns, which will put some order into our code. It should also provide enought flexibility, make the code more testable and reduce code duplication and all other bad software development practices, which normally sprout when one hacks together quickly something.

### OK, I get it, show me the solution
Unfortunately it is not that easy. We need to first look at what already exists out there in order to come up with some ideas and proper architectural design. In the next post I will review some information sources I have found related app architecture. They provide different approaches and present various concepts on how one should build an application architecture. Stay tuned for part 2.
