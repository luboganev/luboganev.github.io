---
layout: post
title: Don't let the scope trick you
description: "This post shows using the let scope function is Kotlin as a flow control structure can lead to unintended bugs."
modified: 2017-07-30
category: post
tags: [android, kotlin, scope, let, software development]
share: true
---

### Kotlin standard library scope functions
The Kotlin language stadard library comes with numerous very useful functions. Probably the most used ones are the scope functions `let`, `with`, `run`, `also`, `apply`. In this post we will focus on a common use case for the function `let` only. For further information about the others please refer to the official library documentation available here: [Kotlin standard library](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/)

### Let's use `let` for nullability checks
This is by far one of the most common uses I have observed in code. I assume that it is partially due to the fact that many mobile developers have tried the Swift language where **optional** types are in the type system just as **nullable** types are for Kotlin. Moreover, the keyword `let` also exists in Swift, and is commonly used to safely **unwrap** optional types and use them if they are not null.

However, the `let` in Kotlin is not a keyword but a higher order function, which brings some benefits but also hidden traps if you use it in a naive way. Let's dive into an example.


### Simple example of nullability handling
Let's assume we have a weather report about current temperature. However, the temperature that could also be null if for example our sensors have not delivered any data yet.

{% highlight kotlin %}
var temperature: Int? = 20

temperature?.let {
  storeInDb(it)
}

{% endhighlight %}

This is a perfectly valid use case for a simple let function usage. We use it all the time and it is totally fine. However, the problems start if you actually need to also perform and action in case the variable has value `null`. Normally in Kotlin, when handling nullability we are use to using the **Elvis operator** `?:`. Let's just naively apply this to this use case. We'll have the following resulting code:

{% highlight kotlin %}
var temperature: Int? = 20

temperature?.let {
  storeInDb(it)
} ?: {
  clearStoredValueFromDb()
}

{% endhighlight %}

Now, at first glance this code looks just fine and logical. It should work, right? 

Well it all depends on what the `storeInDb` is returning... Let me explain. To understand what could go wrong, we need to take a closer look at what the `let` function is actually doing.

{% highlight kotlin %}
inline fun <T, R> T.let(block: (T) -> R): R
{% endhighlight %}

So, the `let` function returns the value, which the passed lambda function return. We know that lambda functions in Kotlin normally have the return value of their last expression. So in our case, this will be the return value of the `storeInDb` function. The problem is, in the naive and quick fix we have done, it is not obvious what this value is, therefore we need to look at the source code of the function `storeInDb`.

{% highlight kotlin %}
fun storeInDb(temperature: Int): Error? {
  try { 
    db.store("temp", temprarature)
    return null
  } catch (e: Exception) {
    return Error("Could not store in the DB.")
  }
}
{% endhighlight %}

So I think by now it would be clear that the fix that we did actually introduced a bug. You see, the `storeInDb` has a return type, which is an Error. When it fails to store the value, it returns an error with a message. However, when it successfully stores the value, it returns null. So how will this result our initial code? Well let's put this all together and see the execution step by step.


{% highlight kotlin %}
fun storeInDb(temperature: Int): Error? {
  try { 
    db.store("temp", temprarature) // 4. The store is successful
    return null // 5. Returning null, since no error happened
  } catch (e: Exception) {
    return Error("Could not store in the DB.")
  }
}

var temperature: Int? = 20 // 1. The value we have is not null

temperature?.let { // 2. Since the value is not null, we invoke the lambda in the let function
  storeInDb(it) // 3. We call the store function. 6. The store function returns `null`
} ?: { 
  // 7. Since the whole previous block returned a null value, we execute the operation inside the elvis block
  clearStoredValueFromDb() // 8. We delete the value we have just stored.
}

{% endhighlight %}

### The solution
It's simple. Just don't use `let` function and `?:` operators for control flow decisions. Just use plain old if else with a temp local variable to allow for smart casing to work in Kotlin, like this:

{% highlight kotlin %}
var temperature: Int? = 20

val temp = temperature
if (temp != null) {
  storeInDb(temp)
} else {
  clearStoredValueFromDb()
}
{% endhighlight %}

It doesn't look any more cluttered than before, but you don't risk having weird bugs. 

### The moral of the story
Be mindful when using Kotlin's amazin language features and rich standard library. Do not fall for the trap for fancy looking or minimalistic code, if it makes the code non-readable or prone to errors.
