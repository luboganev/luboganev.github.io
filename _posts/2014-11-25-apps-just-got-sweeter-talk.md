---
layout: post
title: Apps just got sweeter talk
description: "Apps just got sweeter is a talk I have given at a Google Developer Group Munich meetup. It reports on some insights and hands on experience on the process of redesigning an eclair compatible app to Material Design."
modified: 2014-11-25
category: blog
tags: [android, tip, software development, material design, design, talk]
share: true
---

Today I have given a talk at a meetup of the Google Developer Group in Munich about Material Design. I have shared some insights and hands on experience collected during the process of redesiging one of my private apps - [__AppDetox__](https://play.google.com/store/apps/details?id=de.dfki.appdetox). 

Using the available Android Support Library and some very helpful classes from the Android Support Repository, I was able to convert AppDetox to Material Design. The big win is that I still managed to keep the app available for devices with Android 2.1 Eclair, which used to be rather difficult in the era of Holo Design. Such thing is possible and rather easy to do now thanks to the great Material Design resources provided by Google in the form of extensive documentation and samples. The official Material Design guidelines provided the fundamental understanding of what Material Design is. In addition, the Support Library and the Support Repository provided a lot of out-of-the-box ready UI elements to build a Material Design App. 

By following the new design guidelines, I was also able to update those parts of the user interface, which did not have brand new Material widgets, but still had to be tweaked a bit in order to match the new look. A lot more specific information and examples including links to helpful resources are included in the slides, which you can find here:

[Slides from the talk]({{ site.url }}/documents/2014-11-25-apps-just-got-sweeter-talk/GDG_Munich_20141115_Apps_just_got_sweeter.pdf)