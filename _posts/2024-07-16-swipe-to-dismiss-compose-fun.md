---
layout: post
title: Swipe to dismiss compose fun
description: "This post shows a simple way of implementing swipe to dismiss in Jetpack Compose"
modified: 2024-07-16
category: post
tags: [android, kotlin, jetpack compose, animation, gestures, software development]
share: true
---

# First off existing docs are pretty good
To start off I would like to complement the official documentation provided by Google regarding the drag and swipe gestures in Jetpack Compose. It's quite minimalistic and doesn't dive into any unnecessary details, but instead gets straight to the point. You can find it here: [Drag, swipe, and fling](https://developer.android.com/develop/ui/compose/touch-input/pointer-input/drag-swipe-fling#swiping).

However, every great documentation eventually gets obsolete since software changes. In this case, the whole API was deprecated in favor of a more flexible API that lives in the Compose Foundation layer instead of being only available through Compose Material. Google have provided a great additional article detailing how to migrate from the old APIs to the new ones, also covering advance use cases like additional some customization you might have done. It is available here [Migrate from Swipeable to AnchoredDraggable](https://developer.android.com/develop/ui/compose/touch-input/pointer-input/migrate-swipeable).

The only issue I have with this combination is that if you're a brand new noob like me to the topic of gestures and animations in Compose, you would like instead to have an article that already uses the new APIs instead of following the first one and then migrating. So here we are, this is what this article is about.

# So how do we swipe to dismiss with the new APIs
If you don't feel like reading through the whole post and just wanna get the code, you can find it in this repo: [Swipe to dismiss](https://github.com/luboganev/swipe-to-dismiss-compose). Hopefully, the code is self explanatory.

Anyway, if you want the step by step treatment, here is where we start. What we will build is a Composable that can be swiped horizontally and it moves left or right, fades out while moving towards the final left or right position.

### First define some anchors
First thing we need is to define the so called anchors, which define a map of typed points that correspond to some offset value we will apply to the component to move it around. This is how it looks like

{% highlight kotlin %}
private enum class HorizontalSwipeAnchor {
    LEFT,
    RIGHT,
    CENTER
}

val swipeDistanceDp = 160.dp
val swipeDistancePx = with(LocalDensity.current) { swipeDistanceDp.toPx() }
val anchors = remember {
    DraggableAnchors {
        HorizontalSwipeAnchor.CENTER at 0f
        HorizontalSwipeAnchor.LEFT at swipeDistancePx.unaryMinus()
        HorizontalSwipeAnchor.RIGHT at swipeDistancePx
    }
}

{% endhighlight %}

The keys in the map are the anchors and can be of any type. You aren't forced to create an enum for them - they can be numbers, strings, whatever works for you. The values in that map are the offset values in pixels that correspond to those anchors. Pay attention to the infix function `at`, which is a special DSL for that API. I've mentioned this API feels like a simple map, but there's more to it internally, which we won't dive into right now. 

This article uses a static value for the swipe distance. If you need this to be dynamic, based on layout changes, you will need define the code above to be executed once you know the layout value. This doesn't change much in this code but instead is more relevant for the next part of the code.

### Next up, setup the AnchoredDraggableState
Let's dive straight into the code

{% highlight kotlin %}

val swipeToDismissThreshold = 0.3f
val swipeVelocityPx = with(LocalDensity.current) { swipeDistanceDp.toPx() }
val anchorState = remember {
        AnchoredDraggableState(
            initialValue = HorizontalSwipeAnchor.CENTER,
            anchors = anchors,
            positionalThreshold = { totalDistance -> totalDistance * swipeToDismissThreshold },
            velocityThreshold = { swipeVelocityPx },
            animationSpec = tween()
        )
    }
{% endhighlight %}

In `AnchoredDraggable`, the anchors can be passed to `AnchoredDraggableState`'s constructor or updated through `AnchoredDraggableState#updateAnchors`. Passing the anchors to `AnchoredDraggableState`'s constructor initializes the offset immediately. So if you're in the situation where you need to dynamically calculate the anchors based on layout, you would pick the second option. In this article and the sample code we go for simplicity and work with static values, thus passing the anchors through the constructor.

The `positionalThreshold` parameter to my understanding is the threshold after which the swipe will animate to next matching anchor even if you let it go and don't perform the complete swipe distance.

The `velocityThreshold` parameter to my understanding is the speed threshold of a fling. If you fling faster than that, the swipe will animate to next matching anchor even if you didn't reach the `positionalThreshold` while flinging. In the complete code in the repo I use the same value I calculate for the `swipeDistancePx`, but you can play with that.

### Finally using the state in the anchoredDraggable modifier

To wrap up the MVP, we can put everything together by using the `anchoredDraggable` and the `offset` modifiers like this:

{% highlight kotlin %}

Column(
    modifier = Modifier
        .offset {
            IntOffset(anchorState.offset.roundToInt(), 0)
        }   
        .anchoredDraggable(
            state = anchorState,
            orientation = Orientation.Horizontal,
        )
) {
    ...
}

{% endhighlight %}

What we're doing in the `offset` modifier is using the value provided by the `AnchoredDraggableState` we have initialized in the previous step. The `anchoredDraggable` takes care of everything else related to the gesture handling. So after you apply some basic styling to the Column above you end up with that result.

![Swiping between 3 points]({{ site.url }}/images/2024-07-16-swipe-to-dismiss-compose-fun/step1.gif)

Great. We have swiping with snapping between 3 points. But this isn't really swipe to dimsmiss. However, this is as far as we can get by simply using solely the `AnchoredDraggable` APIs. The following sections add more code to the example to achive the actual swipe to dismiss functionality.

### But now, let's actually create the dismiss effect
Ok, starting with what we have, to simulate a dismiss, we shall do the following additional transformations:

* When the Composable is being swiped towards LEFT or RIGHT, it will fade out and blur a bit for extra style
* We will show the Composable only when the `AnchoredDraggable`'s current state is the CENTER state.
* We will hoist the visibility state of the whole component by having a boolean `show` input parameter as well as an `onDismissRequest` lambda callback, similar to how Compose Dialogs work.

### Adding fade out
With a bit of *simple* calculations, we can create a new derived state based on the `AnchoredDraggable.offset` value. Here is the code:

{% highlight kotlin %}

// Making the component fade out while swiping away toward maximum distance.
// Calculate alpha based on the absolute offset. The value is 1.0f when offset is 0.
// The closer the offset gets to the max swipe distance, the closer this value gets to 0f.
val alpha = remember {
    derivedStateOf {
        abs(abs(anchorState.offset) - swipeDistanceAndVelocityPx)
            .div(swipeDistanceAndVelocityPx)
    }
}

...

Column(
    modifier = Modifier
        // We replaced this with graphicsLayer, since we are modifying also the alpha
        // .offset {
        //    IntOffset(anchorState.offset.roundToInt(), 0)
        // }
        .graphicsLayer {
            this.translationX = anchorState.offset
            this.alpha = alpha.value
        }
        .anchoredDraggable(
            state = anchorState,
            orientation = Orientation.Horizontal,
        )
) {
    ...
}

{% endhighlight %}

Above we create a new `derivedStateOf` for the alpha that changes between 1f and 0f based on the offset from the `AnchoredDraggable`. Notice also that we have replaced the `offset` modifier with the `graphicsLayer`, since we're now changing 2 variables - X translation and alpha value. After doing those steps the results look like that:

![With fading out]({{ site.url }}/images/2024-07-16-swipe-to-dismiss-compose-fun/step2.gif)

### Wrapping up with visibility state hoisting
We will achive this by using 2 different `LaunchedEffects`. Let's take a look at the first one:

{% highlight kotlin %}

LaunchedEffect(key1 = show) {
    when {
        show.not() && anchorState.currentValue == HorizontalSwipeAnchor.CENTER -> {
            anchorState.animateTo(
                targetValue = HorizontalSwipeAnchor.RIGHT,
            )
        }

        show && anchorState.currentValue != HorizontalSwipeAnchor.CENTER -> {
            anchorState.snapTo(HorizontalSwipeAnchor.CENTER)
        }
    }
}

{% endhighlight %}

This launched effect uses the `AnchoredDraggable` methods to either instantly snap its offset to the CENTER value to, or animate it towards the RIGHT value. This achieves the following effect.

![With fading out]({{ site.url }}/images/2024-07-16-swipe-to-dismiss-compose-fun/step3.gif)

The second `LaunchedEffect` takes care of calling the `onDismissRequest` callback, whenever Composable is dismissed through the swipe gesture. It looks like that:

{% highlight kotlin %}

LaunchedEffect(key1 = anchorState.currentValue) {
    when (anchorState.currentValue) {
        HorizontalSwipeAnchor.LEFT, HorizontalSwipeAnchor.RIGHT -> {
            if (show) {
                onDismissRequest()
            }
        }

        HorizontalSwipeAnchor.CENTER -> {
            // Do nothing
        }
    }
}

{% endhighlight %}

The final step is to actually completely remove from the composition the Composable, when it shouldn't be visible. To achieve that we simply wrap it into a check for the `AnchoredDraggable.currentValue` to be equal to CENTER like this:

{% highlight kotlin %}

if (anchorState.currentValue == HorizontalSwipeAnchor.CENTER) {
    Column(
        modifier = Modifier
            ....
    )
}

{% endhighlight %}

And that's it. We have achieved a simple swipe to dismiss functionality with horizontal transition and fade out effect. Now go wild and add more crazy styling and effect to it ;)

P.S. If you've reached that point, I want to thank you for reading all of that. Don't forget to check the full code in the repo here: [Swipe to dismiss](https://github.com/luboganev/swipe-to-dismiss-compose)