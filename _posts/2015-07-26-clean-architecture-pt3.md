---
layout: post
title: Android clean architecture part 3
description: "This post presents a couple of extremely useful articles, blog posts and libraries related to Android app architecture."
modified: 2015-07-26
category: post
tags: [android, MVP, dependecy injection, Dagger, mortar, flow, tip, pattern, architecture, software development]
share: true
---

### Android clean architecture
The first resource I would like to present is a superb blogpost by Fernando Cejas. [Link](http://fernandocejas.com/2014/09/03/architecting-android-the-clean-way/)

It presents an implementation of the Uncle Bob's Clean Architecture in its purest form. The demo application is a simple looking list-detail application. Although it might look an overkill for simplicity of the demo, I could imagine that it can scale really easily to become a feature-rich application, without having to worry about a lot of refactoring or planning. Put in another way, like every architecture, it might seem like an overkill at first, but would eventually pay off in the long run. 

The whole Android Studio project is divided in 3 main project modules -  presentation Layer, domain Layer and data layer.

![Android clean architecture]({{ site.url }}/images/2015-07-26-clean-architecture-pt3/clean_architecture_android.png)

The first layer includes the UI and the Presenters related to it. It is implemented using an MVP pattern.

![Data layer]({{ site.url }}/images/2015-07-26-clean-architecture-pt3/clean_architecture_data.png)

The second layer contains the Interactors implementation and the core business logic of the application.

![Domain layer]({{ site.url }}/images/2015-07-26-clean-architecture-pt3/clean_architecture_domain.png)

The third layer contains the implementation of the data layer. Its implementation is based on the Repository pattern. It encapsulates any reading and writing to cache, disk or network.

![Presentation layer]({{ site.url }}/images/2015-07-26-clean-architecture-pt3/clean_architecture_mvp.png)

Although this implementation of the Clean Architecture is as close as it can get, it is different from the VIPER architecture. In the demo implementation the author has encapsulated, for example, all UI and presenters into one project, whereas all Interactors into another. This is a major difference with VIPER, where a single module responsible for one screen of the application contains all related UI, presenters, interactors and entities. Since we wanted to keep this structure when implementing our Android app, we decided not to base our implementation on this blogpost. In addition, due to the high abstraction of the concrete implementations, it was extremely hard to follow the data flow in the demo app from the UI in one project in one project, through the interactors in another and finally to the network requests or read from disk in the third project. Nevertheless, the demo is an extremely good example of what can be achieved and has provided a lot of takeaways.

### MVP in Android
The second resource I would like to present is from Antonio Leiva. [Link](http://antonioleiva.com/mvp-android/)

In this relatively short blogpost, the author presents the MVP pattern and how it is different from the MVC. The best part is the demo application he provides: [Link](https://github.com/antoniolg/androidmvp)

The demo is very simple to grasp and the introduced pattern is easy to spot there and understand. In addition, the demo application has a VIPER-like structure, where all components (ui, models, presenters and interactors) related to a particular screen are all under the same package name. This makes it really easy to understand and follow the whole data flow. The author also referes to Uncle Bob's clean architecture and how the MVP can be extended to a full implementation of it. He also hints that dependency injection with Dagger could be used for injecting the presenter implementation in the UI instead of the hard-coded instantiating in the demo. This concept grabbed my attention and I have decided to invest some more time into understanding dependency injection as a concept and particularly Dagger. So I focused on what they can contribute to an Android app architecture.

### Dependecy injection with Dagger
There are already a lot of resources available on the subject. You can start by visiting the website of the library here: [Link](http://square.github.io/dagger/). 

However, since Dagger is a pure Java library and is not available only for Android, the examples there are pretty generic and do not help a lot when you try to use it in the context of Android applications.

To the rescue comes again Antonio Leiva with a great series of three blogpost dedicated to using Dagger in Android. What's even better, if you have become familiar with the MVP sample application from him from the previous section, it is the very same demo app which he transforms to use dependency injection with Dagger. This is a highly recommended read you can find here: [Part 1](http://antonioleiva.com/dependency-injection-android-dagger-part-1/), [Part 2](http://antonioleiva.com/dagger-android-part-2/), [Part 3](http://antonioleiva.com/dagger-3/)

### The "Square" architecture
Finally, having read a lot about Dagger, originally developed by Square Inc., I bumped into several blogposts in their tech blog with a lot of critique to the Android SDK and particularly Fragments. I am not a great fan of them either, but what the guys there do is on a whole another level - they advocated against using them altogether and in fact, suggest using a single Activity with many thin Views. A team over there has developed two libraries, which make it possible to implement an Android app with only one activity and only views, which get changed. The libraries are called Mortar and Flow and you can find links to them here: [Mortar](https://github.com/square/mortar), [Flow](https://github.com/square/flow). 

It looks like they would make your life easiear and free from the complicated lifecycle of the activities and the even more chaotic lifecycle of fragments. I'm not going to discuss further this approach, since we have decided against using it. Although it may be a viable solution for Square, we cannot take the risk to base our product on a third party library. For us it would mean escaping from one framework, namely the Android SDK UI stuff, only to land into another, which is unfortunately not familiar to us. I'm not saying it is bad, probably it is awesome to work with, like most of the open source libraries from these guys. We just did not want to invest time and effort into learning this new and still experimental approach.

### Further steps
So after some experimenting, I have finally managed to come up with an "architecture" for our Android application. It is more like a set of rules, which at the end manage to keep a common language across both our iOS and Android apps' code. This solution is based on the MVP similar to the one in the Antonio Leiva's blogposts and uses heavily Dagger for dependency injection, for providing and injecting dependencies, especially while initially connecting all components of a single VIPER module. Implementation details and a sample application will be presented in part 4 of this series. Stay tuned.