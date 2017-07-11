---
layout: post
title: Watch your returns in inline functions
description: "This post discusses a hidden bug in the code related to return statements in lambdas invoked in inline functions"
modified: 2017-07-11
category: post
tags: [kotlin, inline, lambda, return, dangerous, error, software development]
share: true
---

Kotlin is a great programming language and I guess all Android developers have already started adopting it in their code base. However, it has a learning curve and numerous language features, which Java does not have. In order to write bug-free code, we need to be extra careful when using these new features, simply because we might make mistakes due to our years of Java coding. The following example mistake might be an extreme corner case, but it still managed to slip through code review.

### Kotlin inline functions introduction
Kotlin supports higher-order functions, but using them imposes certain runtime penalties: each function is an object, and it captures a closure, i.e. those variables that are accessed in the body of the function. Memory allocations (both for function objects and classes) and virtual calls introduce runtime overhead. Inline functions allow that the function call is effectively replaced by the compiler with the function body. This feature could help with performance improvements, and also can reduce your method count in your codebase, which is especially painful topic in Android development. For more information visit the official documentation for [Kotlin Inline Functions](https://kotlinlang.org/docs/reference/inline-functions.html).

So here is an example of a simple inline function:

{% highlight kotlin %}
inline fun <T, V> executeBetweenWork(param: T, action: (T) -> V): V {
    println("Do work before")
    val result = action(param)
    println("Do work after")
    return result
}
{% endhighlight %}

This function simply prints some text to the console, invokes an input lambda expression and returns its result. Pretty simple and awesome.

### The misplaced return statement
The above function is great, but it could cause trouble if you do not invoke it properly, due to its inline nature. Let me explain by giving an example.

{% highlight kotlin %}
fun wrongCheckSingleTextCharacterWithLambda(): Boolean {
    println("Prepare to invoke inline fun with lambda")
    executeBetweenWork("Some parameter") {
        println("Doing lambda work")
        return it.length > 1
    }
}
{% endhighlight %}

So what exactly is the problem here? Well, the position of the return statement. You see, as Java developers, we're used to writing return statement at the end of a function that returns a value. However, Kotlin will actually show you a compile time error if you try to write a return statements in a lambda.

But wait! Why does the example above even compile? Well, it's due to the fact that the lambda is being passed to an inline function. So when we compile the code, what effectively will happen after the inline function gets replaced is analogous to the following:

{% highlight kotlin %}
fun wrongCheckSingleTextCharacterWithLambda(): Boolean {
    // replaced inline fun body
    println("Prepare to invoke inline fun")
    println("Do work before")
    // replaced labda invocation
    println("Doing Lambda work")
    return "Some parameter".length > 1
    // remaining inline fun body can never be reached
    // and will be removed by the compiler
    // println("Do work after")
    // return result
}
{% endhighlight %}

The code above might not be exactly what Kotlin generates in byte code, but please bear with me, I'm just trying to make a point with it. You see, if we write the return statement at the wrong place we will effectively skip the remaining inline function body, which might be a serious bug and a very hard one to spot. The following example shows the correct way to invoke the lambda in an inline function. Notice the position of the return statement. It is not inside the lambda.

{% highlight kotlin %}
fun correctCheckSingleTextCharacterWithLambda(): Boolean {
    println("Prepare to invoke inline fun with lambda")
    return executeBetweenWork("Some parameter") {
        println("Doing lambda work")
        it.length > 1
    }
}
{% endhighlight %}

And here is the log in the terminal, showing clearly when the "Do work after" is invoked and when not.

{% highlight text %}
=================================================================
Execute function invoking lambda with return in the WRONG place  
=================================================================
Prepare to invoke inline fun with lambda
Do work before lambda
Doing lambda work
=================================================================
Execute function invoking lambda with return in the CORRECT place
=================================================================
Prepare to invoke inline fun with lambda
Do work before lambda
Doing lambda work
Do work after lambda
=================================================================
{% endhighlight %}

### How to not make this mistake
First, just for testing purposes, remove the _inline_ keyword in front of the _executeBetweenWork_ function. When you do this, the compiler will immediately spot the misplaced return statement and mark it as a compiler error. But please do not give up completely on inline functions! There is another way of fixing this issue. You can mark any of the input parameters with the modifier _noinline_ like this:

{% highlight kotlin %}
inline fun <T, V> executeBetweenWork(param: T, noinline action: (T) -> V): V {
    println("Do work before")
    val result = action(param)
    println("Do work after")
    return result
}
{% endhighlight %}

When you do this, the compiler will again spot the misplaced return statement and mark it as a compile time error.

### Summary
Inline functions and lambdas are great features of the Kotlin language. We should definitely learn how to used them and add them to our toolkit for developing apps every day. However, we should be mindful with them and be extra careful to keep our execution flow exactly the way we want it to be.
