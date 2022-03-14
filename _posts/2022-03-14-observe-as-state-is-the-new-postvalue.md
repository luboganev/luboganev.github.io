---
layout: post
title: Observe as state is the new post value
description: "This post discusses a specific behavior of the way Jetpack Compose consumes LiveData"
modified: 2022-03-14
category: post
tags: [android, kotlin, livedata, jetpack compose, software development]
share: true
---

### LiveData - To set value or to postValue?
Before the times of Jetpack Compose, we used to directly observe LiveData for changes in Android components that have lifecycle, such as for example Activity and Fragment. When writing updates to the LiveData, we have two options to choose from - `setValue` and `postValue`. The first one is synchronous operation and has to be executed on the main thread. The second one is asynchronous and can be invoked on other threads. PostValue internally schedules an update to be executed on the main thread Looper for the next Looper cycle, similar to how we can call post or postDelayed on a view.

This behavior is pretty straight forward and essentially results into the following guidelines when to use what:

- If you only care about the last value of a LiveData, you can simply always use postValue. 
- If you care about all intermediate values, then you need to be calling setValue, but then you also need to take care about thread safety yourself.


### Welcome to the future - Jetpack Compose's
Jetpack Compose has its own lifecycle and functions rather differently than what we know from the imperative world of setting view properties by hand. It has its own primitive for observable data called State. In order to glue the LiveData world to the Compose world, we can use a helper extension funciton called [observeAsState](https://developer.android.com/reference/kotlin/androidx/compose/runtime/livedata/package-summary#(androidx.lifecycle.LiveData).observeAsState()). So all is good now, we're all set and can use LiveData the same way as before, right? Not so fast.

### The "issue" with observeAsState
The documentation of the observeAsState() method reads:

> Every time there would be new value __posted__ into the LiveData the returned State will be updated causing recomposition of every State.value usage.

This got me thinking. Is it just English wording or does the word "post" actually convey also a functional meaning here? What happens when we call LiveData.setValue instead of postValue but we use this new function to transform it to Compose State? 

Turns out it is not just a matter of wording. The observeAsState function actually behaves in a way that we will not receive every single intermediate update to the LiveData, regardless of whether we have called setValue or postValue. So it is essentially a forced postValue behavior always. This is a huge difference to what we're used to and can lead to very unexpected bugs especially when we "just" change a view based UI to Compose, without touching the logic in the ViewModel.

### The solution?
There is no silver bullet, since it really depends on your use case. Probably the only time you really care about not missing an update to the LiveData is when you use it to propagate some events or commands, or when you accumulate some data over time. Those are not really the use cases LiveData was created for but hey, it used to work fine before, so why change it :) 

The official Android architecture guidelines explain in great detail how to properly model ui events coming from the ViewModel through LiveData state here: [Handle ViewModel events](https://developer.android.com/jetpack/guide/ui-layer/events?hl=en#handle-viewmodel-events). This will resolve the issue of missing value. However, sometimes implementing this can be quite tricky if your ViewModel is consuming several asynchronous sources of data that utlimately map to UI state changes. It is not trivial to implement logic that waits for the confirmation from the UI that an event has been consumed and only then triggers a LiveData update with some new data that came asynchronously while waiting for the confirmation. 

One solution to resolve the complexity could be to simply split up the UI events into their own dedicated LiveData with the correct confirmation handling as described by the official guide and keep the static UI state data into its own LiveData, where we care only about the last value. 

Alternatively, you could opt out of the official way and simply consume the events LiveData outside of the Compose, so they would behave exactly the way you're used to, given that you produce them by calling setValue.

You can also try to use different type of data channel, such a [Kotlin Flow](https://kotlinlang.org/docs/flow.html) or what is a popular "replacement" for LiveData nowadays - the [SharedFlow or StateFlow](https://developer.android.com/kotlin/flow/stateflow-and-sharedflow). These however, have their own specific behavior which might be what you want or not depending on your use case. I will try to summarize some key aspects of LiveData, StateFlow, SharedFlow in the following table.

| | LiveData         | StateFlow     | SharedFlow |
|-----|--------------|-----------|------------|
| Nullability is defined by the type | No | Yes | Yes |
| Has getter for latest data | Yes | Yes | No |
| Receive latest value immediately when observe/collect | Yes if there is already a value | Yes | No by default, but Yes if replay(1) is added and there was an emission
| Can be initialized without initial value | Yes | No | Yes 
| Backpressure needs handling | No | Yes | Yes
| Each intermediate data update is received when observe / collect | Yes when setValue; No when postValue | No if exactly the same data (equals = true) | Yes |
| Each intermediate data update is received when observeAsState / collectAsState | No | Haven't tested myself. Not sure. | Haven't tested myself. Not sure. |

I would like to apologize for the last two missing cases I didn't have time to test, but feel free to ping me with the results and I'll update the post.

### The moral of the story
Jetpack Compose is an amazing toolkit for building modern UI. However, it has a learning curve that should not be underestimated. The way it works is very different from the classic view based Androiud UI and the glue functions for calling other Jetpack APIs do not necessarily behave exactly how you would expect. Therefore, be sure to always check how something works and don't take your existing knowledge of the current platform as an absolute truth that applies to all new technology you encounter.