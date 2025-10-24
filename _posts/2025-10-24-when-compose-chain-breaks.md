---
layout: post
title: When common refactoring breaks the compose chain
description: "This post highlights a not that obvious error that a common refactoring practice caused when using derivedStateOf"
modified: 2025-10-24
category: post
tags: [android, kotlin, jetpack compose, bug, user error, software development]
share: true
---

# Background story

### How it started
Once upon a time, I had a simple feature in my app that was observing some changing data and just showing it in the UI. For the sake of simplicity, we will model that as showing a simple incrementing counter.

{% highlight kotlin %}

@Composable
fun MainScreen() {
    var counter by remember { mutableIntStateOf(0) }
    Column {

        Text(text = "Counter: $counter")
        
        Button(
            onClick = {
                counter++
            }
        ) { Text(text = "Increment") }
    }
}

{% endhighlight %}

This code worked just fine - whenever the user has clicked the button to increment the counter, the UI reflected the change.

### Adding some more features
I wanted to improve the feature for the users since they only cared if the counter can be divided by 5. Since we no longer need to show the user each change in the UI, in order to reduce the recomposition I used `derivedStateOf` as recommended here: [Use derivedStateOf to limit recompositions](https://developer.android.com/develop/ui/compose/performance/bestpractices#use-derivedstateof). Here is the code:

{% highlight kotlin %}

@Composable
fun MainScreen() {
    var counter by remember { mutableIntStateOf(0) }
    Column {

        val canBeDividedByFive by remember {
            derivedStateOf {
                counter % 5 == 0
            }
        }
        Text(text = "CanBeDividedByFive: $canBeDividedByFive")

        Button(
            onClick = {
                counter++
            }
        ) { Text(text = "Increment") }
    }
}

{% endhighlight %}

This also worked just fine and I was happy with the results. 

### The refactoring that broke the feature
Over time the code of our `Column` grew a lot due to other features being added, so I have decided to extract the feature in its own dedicated composable function. After I have done that, the code looked like this:

{% highlight kotlin %}

@Composable
fun MainScreen() {
    var counter by remember { mutableIntStateOf(0) }
    Column {
        // Imagine code of other features here

        CanBeDividedByFive(counter)

        // and here

        Button(
            onClick = {
                counter++
            }
        ) { Text(text = "Increment") }
    }
}

@Composable
private fun CanBeDividedByFive(counter: Int) {
    val canBeDividedByFive by remember {
        derivedStateOf {
            counter % 5 == 0
        }
    }
    Text(text = "CanBeDividedByFive: $canBeDividedByFive")
}

{% endhighlight %}

I were quite happy with the new structure of the code and the simpler shorter composable functions. However, the feature stopped working, and I had no idea why.

# What is the issue
Jetpack Compose does a lot of things behind the scenes, which we never see in a verbose way in our code. This makes using it really pleasant and simple and leaves the code you write focus on the important stuff. However, the pitfall is that applying common extract function refactoring can break the composable chain and introduce bugs. So without futher due, here is what goes wrong.

{% highlight kotlin %}

@Composable
fun MainScreen() {
    var counter by remember { mutableIntStateOf(0) }
    Column {
        // Imagine code of other features here

        // this still gets called every single time the value changes
        CanBeDividedByFive(counter) 

        // and here

        Button(
            onClick = {
                counter++
            }
        ) { Text(text = "Increment") }
    }
}

@Composable
private fun CanBeDividedByFive(counter: Int) {
    val canBeDividedByFive by remember {
        // this get executed only once when entering the composition
        // and not on changes in counter
        derivedStateOf {
            // this get executed only once when entering the composition
            // and not on changes in counter
            counter % 5 == 0
        }
    }
    Text(text = "CanBeDividedByFive: $canBeDividedByFive")
}

{% endhighlight %}

But why did `derivedStateOf` suddenly lose the ability to react to changes of the `counter`? It worked just fine before and I only refactored it into a function by using the default Injellij refactor extract functionality? What is going on?

To answer this, we need to go back and look at the original code. I will this time only focus on part of the code that is important.

{% highlight kotlin %}

var counter by remember { mutableIntStateOf(0) }
val canBeDividedByFive by remember {
    derivedStateOf {
        counter % 5 == 0
    }
}

{% endhighlight %}

The key here is the way we read `counter` as a delegated property with `by`. In this context, `counter` is a composable `State`, and reading it with delegated property still makes sure the invalidation inside the `derivedStateOf` is triggered. 

However, I broke this observability chain with my refactoring, since we no longer pass to the new function a state, but instead pass in alredy the value of this state.

{% highlight kotlin %}

@Composable
private fun CanBeDividedByFive(counter: Int) // this is not a state but just a snapshot value

{% endhighlight %}

By doing this, `derivedStateOf` never invalidates its internal value, since it seems to treat this value as a constant. If you look at all examples of `derivedStateOf` and its documentation, you won't find any example of using it with something that is not a state, and this is not a conincidence. This is how it works by design and there is nothing wrong with it.

### So how do we fix that?

##### Revert code
No, obviously just kidding. The fact we want to extract the functionality on its own function is not the issue. It is how we're doing it. So let's see some actual possible solutions.

##### Add key to remember of derivedStateOf
This is basically a bandaid. Here is the code.

{% highlight kotlin %}

val canBeDividedByFive by remember(
    key1 = counter // add this as a key of the remember
) {
    derivedStateOf {
        counter % 5 == 0
    }
}

{% endhighlight %}

Why do I say it's a bandaid? Well, it doesn't solve the wrong usage of `derivedStateOf`. And now the feature works, but we're recomposing the new function every time the counter changes, which makes the whole `derivedStateOf` basically useless.

##### Use deferred read through a lambda getter
[Defer reads as long as possible](https://developer.android.com/develop/ui/compose/performance/bestpractices#defer-reads) is a recommended performance optimization for Compose. Using lambda as the function parameter instead of the counter value will fix the issue. This is a better solution, but it could look a bit unreadable. Here is the code:

{% highlight kotlin %}

@Composable
fun MainScreen() {
    var counter by remember { mutableIntStateOf(0) }
    Column {

        CanBeDividedByFive { counter } // lambda here

        Button(
            onClick = {
                counter++
            }
        ) { Text(text = "Increment") }
    }
}

@Composable
private fun CanBeDividedByFive(
    counterProvider: () -> Int // lambda parameter
) {
    val canBeDividedByFive by remember {
        derivedStateOf {
            // calling the lambda parameter
            counterProvider() % 5 == 0 
        }
    }
    Text(text = "CanBeDividedByFive: $canBeDividedByFive")
}

{% endhighlight %}


##### Use state explicitly
The deferred read solution is perfectly fine, but if you prefer to be more explicit and verbose, you should probably just use explicitly `State` as input of the function. Here is the code:

{% highlight kotlin %}

@Composable
fun MainScreen() {
    // notice we changed from var -> val and from by to =
    val counterState = remember { mutableIntStateOf(0) }
    Column {

        CanBeDividedByFive(counterState)

        Button(
            onClick = {
                // notice we changed the mutation here
                counterState.intValue = counterState.intValue + 1
            }
        ) { Text(text = "Increment") }
    }
}

@Composable
private fun CanBeDividedByFive(counterState: IntState) {
    val canBeDividedByFive by remember {
        derivedStateOf {
            // notice we now have to call the value explicitly
            counterState.intValue % 5 == 0
        }
    }
    Text(text = "CanBeDividedByFive: $canBeDividedByFive")
}

{% endhighlight %}

# The moral of the story
Be mindful when using Kotlin and Jetpack Compose amazing features. When refactoring, be careful you don't break any Compose lifecycle, side effects or observability chain. Best is if you have tests that would catch those issues.
