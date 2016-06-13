---
layout: post
title: Alarms and Pending Intents
description: "This post discusses the AlarmManager and the PendingIntents API provided by the Android SDK"
modified: 2016-06-13
category: post
tags: [android, alarm, pending intent, software development]
share: true
---

The Android SDK offers an API for scheduling one time or recurring events called alarms. This is for example how your alarm clock applications work, or how a reminder for a calendar event is triggered. It's pretty simple to use but is has some specific behavior, which you need to take in account. In this post I will describe some of the problems I have encountered when not completely understanding how this API works.

### How do you schedule a simple one time alarm
Well basically it should look something like this.

{% highlight java %}
Calendar c = Calendar.getInstance();
c.add(Calendar.SECOND, 10);
final long afterTenSeconds = c.getTimeInMillis();
final int requestCode = 1;

final Intent myIntent = new Intent("MyIntentAction");
    notificationIntent.addCategory("android.intent.category.DEFAULT");
    notificationIntent.putExtra("message", "Hello world!");
    
AlarmManager alarmManager = (AlarmManager) context.getSystemService(Context.ALARM_SERVICE);
PendingIntent broadcast = PendingIntent.getBroadcast(context, requestCode,
                myIntent, PendingIntent.FLAG_UPDATE_CURRENT);
alarmManager.set(AlarmManager.RTC_WAKEUP, afterTenSeconds, broadcast);
{% endhighlight %}

So what you typically do is to wrap an _Intent_ into a _PendingIntent_ of a particular type and then call the _AlarmManager_ to execute this _PendingIntent_ at some point in time in the future. The last part is, you need to have a component such as a _BroadcastReceiver_, _Activity_ or a _Service_ to receive the wrapped _Intent_ and do something useful with it.

### What is a _PendingIntent_
PendingIntents are a very powerful API in the Android SDK. They allow you to wrap a particular Intent specific to your application in order to give it to another application. The nice thing is, they provide the other application with the required security permissions to actually invoke the Intent you have wrapped. Since Intents are can be broadcasted or used to start an _Activity_ or a _Service_, there are different methods for building a PendingIntent. In the example above, we have used the _getBroadcast()_ method, but there are also methods _getActivity()_ and _getService()_.

What is important to understand is that _PendingIntents_ are not exclusively used only for scheduling alarms through the _AlarmManager_. They are also used for example by the _NotificationManager_ when showing statusbar notifications. Therefore, _PendingIntents_ have a couple of specific properties which you have to understand, before you start using them to schedule alarms.

### The _PendingIntent_ request code
As you have noticed, when building a _PendingIntent_, you are required to provide an integer called _requestCode_. The official documentation states that this is a "private request code for the sender". However, request codes are also a fundamental way to distinguish between _PendingIntent_ instances. You see, the _getBroadcast()_ method is much more than a just a static factory for creating a _PendingIntent_ instance. It internally registers this particular _PendingIntent_ into a kind of global system registry with all _PendingIntents_ from all applications. Remember, the _PendingIntents_ are designed so that other applications could invoke a particular _Intent_ of your application. Therefore a system wide registry makes a lot of sense.

You might wonder, how the single integer for a request code is enough to guarantee that no _PendingIntent_ will be accidentally overwritten by another application? Thankfully, the request code is not solely responsible for determining the uniqueness of a _PendingIntent_.

### _PendingIntent_ disctinction
How do you know if two PendintIntents are the same? It is important to understand that a _PendingIntent_ is just a container with extra functionality, which carries your wrapped _Intent_ inside of it. If you look at the _equals()_ method implementation inside the _PendingIntent_ class, you will find this description:
  
> Comparison operator on two PendingIntent objects, such that true is returned then they both represent the same operation from the same package.  This allows you to use {@link #getActivity}, {@link #getBroadcast}, or {@link #getService} multiple times (even across a process being killed), resulting in different PendingIntent objects but whose equals() method identifies them as being the same operation.

So what exactly does "represent the same operation" mean? I turns out, this is a combination of the request code and the wrapped _Intent_. So now we need to know, how two _Intents_ can be compared. To understand this, you need to look at the documentation of the _Intent.filterEquals()_ method. Basically two Intents are considered equal if their action, data, type, class, and categories are the same. However, nothing is mentioned about their extras _Bundle_. My tests have shown that these extras are completely ignored when comparing _Intents_.

To sum up, you can be quite sure you refer to the same _PendingIntent_ if you know its request code and the action, data, type, class, and categories of the wrapped _Intent_ inside of it.

### _PendingIntent_ cancellation
As I have mentioned above, the _PendingIntent_ is a whole API on its own, completely independent of the _AlarmManager_ or _NotificationManager_. So if there is global registry with all PendingIntents, it makes sense to have some way of clearing them, right? This functionality is provided by the _cancel()_ method of a _PendingIntent_. 

This method can be called only by the application, which originally created the _PendingIntent_. The result of this is that even if another application, such as the AlarmManager, already has a scheduled _PendintIntent_, nothing will happen when it tries to execute it, because it has been cancelled. This is a really powerful functionality to use and I believe that not that many people are actually aware of it.

### _PendingIntent_ flags
You might have also noticed the _PendingIntent.FLAG_UPDATE_CURRENT_ parameter used when calling the _getBroadcast()_ method. The last parameter of the method is an integer containing a bitmask of _PendingIntent_ flags. These flags are also really crucial to understand when you use the PendingIntent API. There are just a few flags you can use:

* FLAG_ONE_SHOT
* FLAG_NO_CREATE
* FLAG_CANCEL_CURRENT
* FLAG_UPDATE_CURRENT
* FLAG_IMMUTABLE

I will not go into much detail about all different flags, since their documentation is really self-explanatory. I will focus only on one of them, since it really nicely gives an insight into what acutally happens when you call the _getBroadcast()_, _getService()_ or _getActivity()_ method of the _PendingIntent_ class 

#### The flag _FLAG_NO_CREATE_.

Its documentation states: 

>Flag indicating that if the described PendingIntent does not already exist, then simply return null instead of creating it. For use with {@link #getActivity}, {@link #getBroadcast}, and {@link #getService}.

So when you look at all other flags and this one, you realize that every time you call _getBroadcast()_, _getService()_ or _getActivity()_, you put or update a specific _PendingIntent_ into the global registry. _FLAG_NO_CREATE_ is the only way to call any of these methods, without actually registering anything new. So the question is, when would you want to use this? This flag is really useful if you just want to check if a particular _PendingIntent_ exists and is active. The following code snippet illustrates such sample usage.

{% highlight java %}
final int requestCode = 1;

final Intent myIntent = new Intent("MyIntentAction");
    notificationIntent.addCategory("android.intent.category.DEFAULT");
    notificationIntent.putExtra("message", "Hello world!");

PendingIntent broadcast = PendingIntent.getBroadcast(context, requestCode,
                myIntent, PendingIntent.FLAG_NO_CREATE);
if (broadcast != null) {
    broadcast.cancel();
}

{% endhighlight %}


### Cancelling alarms
Now that we understand how _PendingIntents_ work, we can actually look at the _AlarmManager_ API again. It provides a method for cancelling scheduled alarms, which expects a _PendingIntent_ instance as a parameter. The documentation of this method is unfortunately not complete. It states the following:

>Remove any alarms with a matching {@link Intent}. Any alarm, of any type, whose Intent matches this one (as defined by {@link Intent#filterEquals}), will be canceled.

One can assume from it that as long as the wrapped _Intent_ matches, you have cancelled the alarm, right? Unfortunately this is a wrong assumption!

As we know from the _PendingIntent_ matching described above, only if both the request code and the wrapped _Intent_ match, then we can be sure that we are referring to the same _PendingIntent_. Thus we will cancel exactly the alarm related to the input _PendingIntent_ parameter only if both its request code and wrapped _Intent_ match. The following snippet illustrates this behavior:

{% highlight java %}
Calendar c = Calendar.getInstance();
c.add(Calendar.SECOND, 10);
final long afterTenSeconds = c.getTimeInMillis();
final int requestCode = 42;
final int anotherRequestCode = 1337;
final AlarmManager alarmManager = (AlarmManager)
    context.getSystemService(Context.ALARM_SERVICE);
final Intent sameIntent = new Intent("MyIntentAction");
    notificationIntent.addCategory("android.intent.category.DEFAULT");
    notificationIntent.putExtra("message", "Hello world!");

final PendingIntent broadcast = PendingIntent.getBroadcast(context, 
    requestCode, sameIntent, PendingIntent.FLAG_UPDATE_CURRENT);
                
alarmManager.set(AlarmManager.RTC_WAKEUP, afterTenSeconds, broadcast);

// Check PendintIntent exists
assertTrue(PendingIntent.getBroadcast(context, requestCode,
                sameIntent, PendingIntent.FLAG_NO_CREATE) != null));
    
alarmManager.cancel(PendingIntent.getBroadcast(context, anotherRequestCode, 
    sameIntent, PendingIntent.FLAG_UPDATE_CURRENT));
    
// Check PendintIntent exists
if (PendingIntent.getBroadcast(context, requestCode,
                sameIntent, PendingIntent.FLAG_NO_CREATE) != null)) {
    // We just cancelled the same Intent, WTF?!
}

// Try cancel with the same request code
alarmManager.cancel(PendingIntent.getBroadcast(context, requestCode, 
    sameIntent, PendingIntent.FLAG_UPDATE_CURRENT));
    
// Check PendintIntent exists
if (PendingIntent.getBroadcast(context, requestCode,
                sameIntent, PendingIntent.FLAG_NO_CREATE) == null)) {
    // OK, we have successfully cancelled the alarm
}

{% endhighlight %}

Another important thing to note is that the _AlarmManager.cancel()_ method does not cancel the _PendingIntent_ itself. So even after calling this method, the initial _PendingIntent_ is still active in the registry. However, this should be expected, since as already mentioned, the _PendingIntents_ are not used only by the _AlarmManager_, but are independent API in the SDK.

### AlarmManager and PendingIntent lifecycle
A very important thing to keep in mind is that alarms and _PendingIntents_ are not always kept by the framework. I have performed some tests to try to find out when exactly are alarms being disabled and when exactly are _PendingIntents_ being cancelled, depending on external influence by the framework or the user. The following results have been consistent on three different devices running three different Android versions.

* Motorola Nexus 6 running Android 6.0
* LG G3S running Android 4.4
* Sony Xperia Z1 Compact running Android 5.1

### PendingIntent & AlarmManager lifecycle test results

| Performed action |&nbsp;&nbsp;| Cancel PendingIntent |&nbsp;&nbsp;| Clear alarm |
|-------------------------------|----------------------|-------------|
| Force close from app settings || no || yes |
| Device restart || yes || yes |
| Runtime exception || no || no |
| ANR Force close || no || no |
| Swipe away from recent apps || no || no |
| Clear app data from settings || yes/no || yes |

As you can see, Alarms are not 100% reliable and there are a lot of situations, where you need to take care and reschedule them. The only situation you cannot do anything about it, is when the user Force closes your application from the app settings screen. If this occurs, the Android framework will NOT invoke automatically any _Service_ or _BroadcastReceiver_ of your application unless the user explicitly starts your application again. This might look a little harsh for us developers, but makes perfect sense from a user point of view. Imagine some _Service_, which crashes immediately every time it gets started. If it is a sticky service, the framework will try to restart it over and over again. Constantly popping an application not responding dialog in front of the user will prevent normal device usage, which is really bad. So this fail-safe functionality is actually a good thing for the whole system and for other apps.

### Time to play - try out the different options in the companion app
I have included a demo in the blog's companion app where you can test some _PendingIntents_ and alarm scheduling. You can find the app and its source code under the following links:

* [**Testground at Google Play**](https://play.google.com/store/apps/details?id=com.luboganev.testground)
* [**Testground source code at GitHub**](https://github.com/luboganev/testground)