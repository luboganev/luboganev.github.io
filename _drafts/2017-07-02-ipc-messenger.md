---
layout: post
title: IPC with Messenger
description: "This post shows how to implement inter-process communication in Android by using the Messenger API provided by the Android SDK"
modified: 2017-07-02
category: post
tags: [android, kotlin, ipc, bound service, service, messenger, software development]
share: true
---

### Time to play - try out the different options in the companion app
I have included a demo in the blog's companion app where you can test the _Messenger IPC_, including two types of remote service calls:

* Fire-and-forget - Call, which sends a _String_ message to the service and expects no response
* Request-Response - A call to the service sending two integers, which expects a response with the result of their addition. In addition, the integers are packed into a custom data class, in order to demonstrate the ability to send _Parcelable_ payload through the Messenger API.

You can find the app and its source code under the following links:

* [**Testground at Google Play**](https://play.google.com/store/apps/details?id=com.luboganev.testground)
* [**Testground source code at GitHub**](https://github.com/luboganev/testground)
