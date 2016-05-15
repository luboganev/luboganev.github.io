---
layout: post
title: Touch stealing wrapper
description: "In this post we will look into a simple wrapper layout which helps managing touch events passed to its children"
modified: 2014-10-11
category: blog
tags: [android, tip, software development, touch, wrapper]
share: true
---

A couple of weeks ago I have been developing a new feature at my job, which included a lot of user interaction with a map. Althoug the MapView class provided with the Google Play Services SDK is really nice and easy to use, I had a case, which required me to do a very specific thing - get notified when a user interacts with the map itself without actually allowing touch events to go through in some occasions. It was crucial that I get notified before the touch event reaches the MapView itself.

### The dispatchTouchEvent method coming to the resque
After some searching for a solution, I have found a specific method called **dispatchTouchEvent**. It is a method belonging to the **ViewGroup** base class ([link](http://developer.android.com/reference/android/view/ViewGroup.html#dispatchTouchEvent\(android.view.MotionEvent\))). It passes the touch screen motion event down to a target view, or the current view if it is the target, which is exactly what I needed. I could simply override this method and do all the checks I needed there before I allow the touch to reach the view itself. Luckily, the MapView class actually extends FrameLayout, which extends the ViewGroup base class, so it was an easy win for me. 

### A simple wrapper layout
Although similar method is provided in every standard View or ViewGroup from the Android SDK, you sometimes might not want to extend the thing you want to control into a custom class encapsulating this functionality. Sometimes you also might not be able to, if the View or ViewGroup is declared final and you cannot extend it. In such cases, you could just wrap such view into a simple container where you do all your magic. The following example shows a simple FrameLayout view that does just that. All the magic happens inside the **dispatchTouchEvent** method.

{% highlight java %}
@Override
public boolean dispatchTouchEvent(MotionEvent ev) {
    switch (ev.getAction()) {
        case MotionEvent.ACTION_DOWN:
            mIsTouched = true;
            if(mOnTouchDownListener != null) 
                mOnTouchDownListener.onTouchDownStateChanged(mIsTouched);
            break;
        case MotionEvent.ACTION_UP:
            mIsTouched = false;
            if(mOnTouchDownListener != null) 
                mOnTouchDownListener.onTouchDownStateChanged(mIsTouched);
            break;
    }

    if(mTouchBlocking) {
        return true;
    }
    return super.dispatchTouchEvent(ev);
}
{% endhighlight %}

A simple callback interface is there, which will signal to you whether a touch event occurs, before it reaches the view itself, since the super method is called at the very end. Of course, you might want to block completely this touch event. To do so, you just have to steal it away by returning true instead of calling the super method. Below you can find the full source code of this simple wrapper.

{% highlight java %}
public class TouchDownWrapper extends FrameLayout {
    private boolean mIsTouched = false;
    private boolean mTouchBlocking = false;

    public TouchDownWrapper(Context context) {
        super(context);
    }

    public TouchDownWrapper(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    public TouchDownWrapper(Context context, AttributeSet attrs, int defStyle) {
        super(context, attrs, defStyle);
    }

    @Override
    public boolean dispatchTouchEvent(MotionEvent ev) {
        switch (ev.getAction()) {
            case MotionEvent.ACTION_DOWN:
                mIsTouched = true;
                if(mOnTouchDownListener != null) 
                    mOnTouchDownListener.onTouchDownStateChanged(mIsTouched);
                break;
            case MotionEvent.ACTION_UP:
                mIsTouched = false;
                if(mOnTouchDownListener != null) 
                    mOnTouchDownListener.onTouchDownStateChanged(mIsTouched);
                break;
        }

        if(mTouchBlocking) {
            return true;
        }
        return super.dispatchTouchEvent(ev);
    }

    private OnTouchDownStateChangedListener mOnTouchDownListener;

    public void setOnTouchDownStateChangedListener(
                    OnTouchDownStateChangedListener listener) {
        mOnTouchDownListener = listener;
    }

    public boolean isTouched() {
        return mIsTouched;
    }

    public boolean isTouchBlockingEnabled() {
        return mTouchBlocking;
    }

    public void setTouchBlockingEnabled(boolean touchBlocking) {
        this.mTouchBlocking = touchBlocking;
    }

    public static interface OnTouchDownStateChangedListener {
        public void onTouchDownStateChanged(boolean isTouched);
    }

}
{% endhighlight %}

### Time to play - try out the different options in the companion app
I have included a demo in the blog's companion app where you can test everything you would like. You can find the app and its source code under the following links:

* [**Testground at Google Play**](https://play.google.com/store/apps/details?id=com.luboganev.testground)
* [**Testground source code at GitHub**](https://github.com/luboganev/testground)