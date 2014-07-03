---
layout: post
title: Permissions, features and Google Play
description: "This post discusses a programming error which can influence the way Google Play filters the supported devices for your app."
modified: 2014-06-22
category: blog
tags: [android, google play, tip, permissions, features]
share: true
---

I have been dealing with adding extra features and permissions to an app I have worked on lately. Through the process I bumped into 
many problems since I had not fully understood a couple of concepts related to them. I would like to share some of them with you so that
you do not miss them when reading the official documentation.

### Developing mobile apps is complex
As mobile app engineers we always focus pretty much on the technical part, i.e. the coding of the mobile app according 
to all guidelines, requirements, etc. Our main goal is to provide a bug-free, fast nice looking app. This is not a trivial task at all 
and sometimes if we need to use some of the features of the devices such as the camera or the bluetooth or the gps, we might 
forget to properly setup all required tweaks. We know we have to add the necessary permission to the app's manifest file and if we
don't do it, the app will crash. This is a pretty obvious problem which we can spot quickly and fix it before releasing. 

But most of the time that's really our main concern - implement and test all possible conditions on various devices and configurations. 
At the end we are proud because we have created a robust app which works fine on every possible device. We have even made is so well 
that it does not crash on devices which completely lack some hardware such as the camera or the gps. However, there is still one thing 
we might have missed to setup properly which affects directly the number of users who will be able to install our app - Google Play 
device filtering.

## Permissions, features and Google Play device filtering
Let's say we're building an app which can take pictures with the camera. The only thing we really need to do in order to make it work 
is adding the `<uses-permission android:name="android.permission.CAMERA" />` to the manifest right? Well, not exactly. 
The official [**Camera Manifest**](http://developer.android.com/guide/topics/media/camera.html#manifest) section in the 
documentation provides you with a lot of information on the topic.

> Camera Features - Your application must also declare use of camera features. Adding camera features to your manifest causes 
Google Play to prevent your application from being installed to devices that do not include a camera or do not support the camera 
features you specify. If your application can use a camera or camera feature for proper operation, but does not require it, you should specify this in the manifest by including the android:required attribute, and setting it to false.

It is pretty clear, right, except for one corner case. We all make mistakes, let's say you simply forget to declare the features in your
manifest file. What happens then? This case is called implicit feature and is described in the [**Filtering based on implicit features**](http://developer.android.com/guide/topics/manifest/uses-feature-element.html#implicit) section in the documentation.

>An implicit feature is one that an application requires in order to function properly, but which is not declared in a <uses-feature> element in the manifest file. Strictly speaking, every application should always declare all features that it uses or requires, so the absence of a declaration for a feature used by an application should be considered an error. However, as a safeguard for users and developers, Google Play looks for implicit features in each application and sets up filters for those features, just as it would do for an explicitly declared feature.

So after we're familiar with how things work let's focus on our example. We have simply declared the camera permission in our manifest and completely forgotten declaring any features. According to the table with [**Permissions that Imply Feature Requirements**](http://developer.android.com/guide/topics/manifest/uses-feature-element.html#permissions), our application additionally declares the following implicit features: `android.hardware.camera` and `android.hardware.camera.autofocus`. 

Now, this means that no device on the Play Store which does not have a camera will be able to find our app or install it. This is not that bad, when the main functionality of the app is taking pictures. However, it is bad for apps which may offer taking pictures as an extra
feature which only complements the main app functionality.

Think of a messenger app. The main feature is to send around text messages. Then we include an additional sending picture feature through some custom camera implementation in the app. If we forget to declare the camera features as optional, no device without a camera will be able to install our app in the future, thus we will lose users with no camera who simply want to text, just because we made this stupid error. When publishing a release, Google Play Developer Console will notify us about the number of supported devices, so pay attention there for any excluded devices. If you notice that there are a lot of unsupported devices, go back to your manifest and check if you have done everything properly.