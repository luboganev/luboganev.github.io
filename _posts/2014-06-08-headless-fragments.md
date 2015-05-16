---
layout: post
title: Headless fragments
description: "This post discusses what headless fragments are and proposes one use of them"
modified: 2014-06-08
category: blog
tags: [android, tip, software development, fragment, asynctask, orientationchange]
share: true
---

When Fragments were introduced a couple of years ago, they were presented as reusable UI compontents which could co-exist in the same Activity. They also provide features related to backstack, which were not available before when people did the same thing using fat custom view. I will not go into much detail about all the features Fragments provide for building a dynamic and modern App UI. In this post I will focus on another property of Fragments, which many people are not familiar with, or just forget when they build Apps.

###What are Headless Fragments?
I have no idea whether it is an official name or it originates from the developers community, anyway, it pretty much describes what they are. Headless Fragments are Fragments which do not have any UI, i.e. they do not inflate any XML resource or create and return a View. In terms of MVC(Model View Controller), they contain only the Controller part without actually be responsible directly for controlling a View. They are instantiated only programmatically and added to Activities by calling the methods of the FragmentManager. More on this later.

###A very useful feature of Headless Fragments
Headless Fragments, have one really useful feature - they can be retained by the FragmentManager across configuration changes. Since they do not have any UI related to them, they do not have to be detroyed and rebuilt again when the user rotates the device for example. In order to activate this behaviour, one just has to set the retained flag of the Fragment when it is initialized. This can be done in the **onCreate** method of the Fragment.

{% highlight java %}
@Override
public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setRetainInstance(true);
}
{% endhighlight %}

###Managing Headless Fragments
As I have already mentioned, Headless Fragments are created purely programmatically. Since they do not have any UI, there is no resource id from a XML related to them. Fortunately, the FragmentManager class offers alternative methods for managing such Fragments based on the so called **Tag**, which is just a String. Take in account, it is our job to make sure these Tags are unique for Headless Fragments inside an Activity. The following code snippet demostrates how a Headless Fragment can be initialized and added to an Activity (for example in its **onCreate** method). Notice that we first check if the Fragment is already available by searching for it using the Tag. We create it only if no Fragment with the requested Tag is found by the FragmentManager.

{% highlight java %}
HeadlessCounterFragment counterState = 
    (HeadlessCounterFragment)getFragmentManager()
        .findFragmentByTag("counter_fragment");
        
if(counterState == null) {
    counterState = new HeadlessCounterFragment();
    getFragmentManager().beginTransaction()
        .add(counterState, "counter_fragment").commit();
}
{% endhighlight %}

###Some uses cases
So what could we use this type of Fragments for? It's simple, they do not get detroyed, therefore one can put inside whatever object needed and it will stay there even across configuration changes. For simpler objects this is an overkill and I recommend just keeping the state by implementing the **onSaveInstanceState** and **onRestoreInstance** methods. However, not everything can be implemented in a way that it can fit into a Bundle. Some classes are just too complicated to be implemented as a Parceable. Sometimes one simply does not want to write all the extra boilerplate code needed for it. Or sometimes one wants a simple AsyncTask to do something and keep doing it across orientation change. In such cases, one can simply save a reference to the needed objects inside a Headless Fragment and can be sure that whenever he needs this reference, it will be there because the Fragment will not be destroyed. 

Now, Headless Fragments are not some magic solution for everything and should be used with caution. I'm not advocating doing really long operations like downloading files inside a Headless Fragment. One should use some proper background functionality like a long running Service for that. In addition, one has to be careful with saving references into a Headless Fragments since they potentially have longer lifecycle than other UI components. For example, doing something stupid like saving a reference to a whole Activty could cause a memory leak on orientation change. My point is, be mindful about what you use Headless Fragments for.

###A simple example
The following code snippets represent a simple example usage of Headless Fragment. The Fragment contains an AsyncTask which just counts up starting from one until it is stopped. The Fragment also contains a reference to a TextView in the UI which is updated by the counter. You will notice that I have implemented the reference to the TextView as a WeakReference, which allows the TextView instance to be freed by the Garbage Collector if neessary, thus will not produce any memory leaks. 

There is also some logic for terminating the AsyncTask counter if the TextView is gone. This should normally not happen, but if you want to test it, you have to make the following:

1. Make sure you call `counterState.setCounterTextView()` in `onCreate()` only once when the parent Activity is created and then do not call it if it gets recreated due to orientation change. 
2. Depending on your device, you might want to rotate the device a couple of times before the Garbage Collector kicks in.

The first code snippet represents the implementation of the Fragment.
{% highlight java %}
public static class HeadlessCounterFragment extends Fragment {
    private SimpleCounterAsync mSimpleCounterAsync = new SimpleCounterAsync();
    private WeakReference<TextView> mCounterTextView;

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setRetainInstance(true);
    }

    public void setCounterTextView(TextView textView) {
        mCounterTextView = new WeakReference<TextView>(textView);
    }

    public void startCounting() {
        if(mSimpleCounterAsync.getStatus() == AsyncTask.Status.RUNNING) return;
        if(mSimpleCounterAsync.getStatus() == AsyncTask.Status.FINISHED)
            mSimpleCounterAsync = new SimpleCounterAsync();
        mSimpleCounterAsync.execute();
    }

    public void stopCounting() {
        if(mSimpleCounterAsync.getStatus() != AsyncTask.Status.RUNNING) return;
        mSimpleCounterAsync.cancel(true);
    }

    private class SimpleCounterAsync extends AsyncTask<Void, Integer, Void> {
        private int mViewIsGoneCount = 0;

        @Override
        protected Void doInBackground(Void... voids) {
            int count = 0;

            while(!isCancelled()) {
                count++;
                publishProgress(count);
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    return null;
                }
            }
            return null;
        }

        @Override
        protected void onProgressUpdate(Integer... values) {
            super.onProgressUpdate(values);
            TextView view = mCounterTextView.get();
            if(view != null) {
                view.setText("Counter: " + values[0]);
                mViewIsGoneCount = 0;
            } else {
                mViewIsGoneCount++;
                if(mViewIsGoneCount < 5) {
                    Toast.makeText(getActivity(), "TextView is not there for "
                            + mViewIsGoneCount + " time.", 
                                Toast.LENGTH_SHORT).show();
                } else {
                    Toast.makeText(getActivity(), "TextView was gone for too long. " +
                            "Terminating the counter.", 
                                Toast.LENGTH_SHORT).show();
                    cancel(true);
                }
            }
        }
    };
}
{% endhighlight %}

The following snippet contains the rest of the code of the Activity where the HeadlessConterFragment is hosted.
{% highlight java %}
private HeadlessCounterFragment mHeadlessCounterFragment;

@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_headlessfragmentdemo);

    mHeadlessCounterFragment = (HeadlessCounterFragment)getFragmentManager()
            .findFragmentByTag("counter_fragment");
    if(mHeadlessCounterFragment == null) {
        mHeadlessCounterFragment = new HeadlessCounterFragment();
        getFragmentManager().beginTransaction().add(mHeadlessCounterFragment, "counter_fragment").commit();
    }

    if(savedInstanceState == null) {
        // Setting the TextView for the count only initially
        mHeadlessCounterFragment.setCounterTextView((TextView) findViewById(R.id.textView));
    }

    findViewById(R.id.btn_startCounting).setOnClickListener(new View.OnClickListener() {
        @Override
        public void onClick(View view) {
            HeadlessCounterFragment counterState = (HeadlessCounterFragment)getFragmentManager()
                    .findFragmentByTag("counter_fragment");
            if(counterState != null) counterState.startCounting();
        }
    });

    findViewById(R.id.btn_stopCounting).setOnClickListener(new View.OnClickListener() {
        @Override
        public void onClick(View view) {
            HeadlessCounterFragment counterState = (HeadlessCounterFragment)getFragmentManager()
                    .findFragmentByTag("counter_fragment");
            if(counterState != null) counterState.stopCounting();
        }
    });
}

@Override
protected void onDestroy() {
    super.onDestroy();
    // Making sure we clean references on destroy
    if(mHeadlessCounterFragment != null) {
        mHeadlessCounterFragment.setCounterTextView(null);
        mHeadlessCounterFragment = null;
    }
}
{% endhighlight %}

Finally, the following snippet contains the XML layout file which is inflated by the Activity.

{% highlight xml %}
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context="com.example.blogpostsplayground.app.AwsumActivity">

    <TextView
        android:text="@string/hello_world"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:id="@+id/textView" />

    <LinearLayout
        android:orientation="horizontal"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_below="@+id/textView"
        android:layout_marginTop="8dp">

        <Button
            android:id="@+id/btn_startCounting"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Start counting"/>

        <Button
            android:id="@+id/btn_stopCounting"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Stop counting"/>

    </LinearLayout>

</RelativeLayout>
{% endhighlight %}

###There is always a library for that
The Android development community is awesome and there are many really talented developers who open-source fantastic libraries tackling common problems. The solution described in this post is by no means exclusively my idea. There is already an open-source library which offers the nice features of Headless Fragments for storing objects across orientation changes. It is called **Memento** and you can find it on [GitHub](https://github.com/mttkay/memento). It is really easy to use and hides all the boilerplate code related to the HeadlessFragment by employing Annotations. 

Now, why did I keep this information till the very last moment and didn't just give you the nice library at the very beginning? Well, I wanted you to understand how such libraries are possible and what principles are they based on. I personally hate it when I have no idea how something that I use works, because it looks like a black magic box and I have no idea what to expect from it.