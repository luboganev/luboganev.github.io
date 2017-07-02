---
layout: post
title: Bound services with Messenger
description: "This post shows how to implement inter-process communication in Android by using the Messenger API provided by the Android SDK"
modified: 2017-07-02
category: post
tags: [android, kotlin, ipc, bound service, service, messenger, software development]
share: true
---

### Android inter-process communication (IPC)
TL;DR; if you want to get a very good understanding into how Android inter-process communication works, check out the following outstanding video about the Android Binder framework internals. It is long, but I really recommend it!

* [AnDevCon IV: Android Binder IPC Framework](https://www.youtube.com/watch?v=hiq3mGfLOtE)

> Inter Process Communication (IPC) has been a part of Android since 1.0, and yet most of us take it for granted. Intents, content providers, and system service managers hide the IPC infrastructure provided by Binder, but without it, the Android OS and our apps would simply fall apart. Binder/IPC is the glue that holds it all together. It enables Android's memory management, security sandboxing, efficient threading, and countless other features on the Android platform.

### What are bound services
A bound service is the server in a client-server interface. It allows other components to bind to the service, send requests, receive responses, and perform interprocess communication (IPC). Multiple clients can connect to a service simultaneously. A bound service typically lives only while it serves another application component and does not run in the background indefinitely, but an indefinitely running service can also be a bound to. For more information on service lifecycle, please refer to the official Android documentation about [Managing the lifecycle of a bound service](https://developer.android.com/guide/components/bound-services.html#Lifecycle).

### How to bind to a service
There are 3 different ways to bind to a Service:

* Extending the Binder class
* Using a Messenger
* Using AIDL

The following tables summarize the most important properties of these very different communication channels:

Client & service location | Binder class | Messenger | AIDL
--|---|---|---|--
Same app and same process  |YES|YES|YES
Same app but different process  |NO|YES|YES
Different app  |NO|NO|YES

 | Binder class | Messenger | AIDL
--|---|---|---|--
Threading setup of client & service  |same single thread (main thread)|Single thread with message queue|Fully multi-threaded

### Bound services with Messenger
[Official documentation](https://developer.android.com/guide/components/bound-services.html#Messenger)

##### We will need the following classes

* [Service](https://developer.android.com/reference/android/app/Service.html) - A Service is an application component representing either an application's desire to perform a longer-running operation while not interacting with the user or to supply functionality for other applications to use. In our case we need to extend this class in order to implement a bound service.
* [ServiceConnection](https://developer.android.com/reference/android/content/ServiceConnection.html) - this interface is used by a client in order to establish connection with a bound service. It gets callbacks from the OS, when the client connects to a service or when the connection is lost.
* [Messenger](https://developer.android.com/reference/android/os/Messenger.html) - This class represents a reference to a Handler, which others can use to send messages to it. This allows for the implementation of message-based communication across processes, by creating a Messenger pointing to a Handler in one process, and handing that Messenger to another process. In our case, when connected to a bound service, the clients receive a Messenger initialized by that service. They can use it to send IPC calls to the bound service. If they want to receive responses back from the bound service, then they need to initialize their own Messenger, and pass it via the [replyTo](https://developer.android.com/reference/android/os/Message.html#replyTo) field of the messages sent to the bound service through the bound service Messenger. This way, when the bound service receives a Message containing a replyTo Messenger, it could extract this Messenger and send the response messages through it.
* [Handler](https://developer.android.com/reference/android/os/Messenger.html) - A Handler allows to send and process Message objects associated with a thread's MessageQueue. Each Handler instance is associated with a single thread and that thread's message queue. In our case, the service needs to initialize a Handler, wrap it into a Messenger and provide this Messenger to connected clients so that they can send messages. The incoming messages can be processed in the [handleMessage](https://developer.android.com/reference/android/os/Handler.html#handleMessage(android.os.Message)) method of the Handler.
* [Message](https://developer.android.com/reference/android/os/Message.html) - Defines a message containing a description and arbitrary data object that can be sent to a Handler. In our case, client requests to the bound service and service responses will be wrapped into instances of the Message class and passed through the corresponding Messenger.
* [Intent](https://developer.android.com/reference/android/content/Intent.html) - An intent is an abstract description of an operation to be performed. In our case it is used when clients want to bind to a bound service. It contains the necessary information, so that the OS can locate the application and component, where the implementation of the bound service is done. On the service side, when client binds, the incoming Intent can be used to identify the connecting client.
* [Bundle](https://developer.android.com/reference/android/os/Bundle.html) - A mapping from String keys to various [Parcelable](https://developer.android.com/reference/android/os/Parcelable.html) values. In our case, Bundles are used to define the properties of the bind Intent constructed by the client. In addition, all data in Message sent back and forth between client and service is a Bundle.

##### Some observations

When Messengers are used for bound service communication, there are no explicit calls to particular service methods, instead Messages have to include enough payload and metadata to uniquely define the operation which the service should perform.

Further, there are no restrictions for the number of Messenger-Handler pairs on each side of the communication, but the single threaded behavior and order of handled messages is guaranteed only for each of these pairs. If multi-threaded approach needed, then implementing different Messenger-Handler for each client connection is not the way to go. [AIDL](https://developer.android.com/guide/components/aidl.html) should be used instead.

Finally, Messages are very generic objects, therefore the service and the client have to share a common contract for defining the available commands, flags, payload formatting, and common Parcelable data classes. This contract has to be shared between the client and service, which means a shared library in case they are located in different apps.

##### Defining the common contract

##### Building the service part

##### Building the service part

### Time to play - try out the different options in the companion app
I have included a demo in the blog's companion app where you can test the _Messenger IPC_, including two types of remote service calls:

* Fire-and-forget - Call, which sends a _String_ message to the service and expects no response
* Request-Response - A call to the service sending two integers, which expects a response with the result of their addition. In addition, the integers are packed into a custom data class, in order to demonstrate the ability to send _Parcelable_ payload through the Messenger API.

You can find the app and its source code under the following links:

* [**Testground at Google Play**](https://play.google.com/store/apps/details?id=com.luboganev.testground)
* [**Testground source code at GitHub**](https://github.com/luboganev/testground)
