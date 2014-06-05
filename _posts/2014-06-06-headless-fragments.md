---
layout: post
title: Headless fragments
description: "This post discusses what headless fragments are and proposes one use of them"
modified: 2014-06-06
category: blog
tags: [android, tip, software development, fragment, asynctask, orientationchange]
share: true
---

[Memento](https://github.com/mttkay/memento)

{% highlight java %}
@Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setRetainInstance(true);
    }
{% endhighlight %}

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

{% highlight java %}
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_awsum);
    
    HeadlessCounterFragment counterState = 
        (HeadlessCounterFragment)getFragmentManager()
            .findFragmentByTag("counter_fragment");
    if(counterState == null) {
        counterState = new HeadlessCounterFragment();
        getFragmentManager().beginTransaction()
            .add(counterState, "counter_fragment").commit();
    }

    counterState.setCounterTextView((TextView)findViewById(R.id.textView));

    findViewById(R.id.btn_startCounting)
        .setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                HeadlessCounterFragment counterState = 
                    (HeadlessCounterFragment)getFragmentManager()
                        .findFragmentByTag("counter_fragment");
                if(counterState != null) counterState.startCounting();
            }
    });

    findViewById(R.id.btn_stopCounting).setOnClickListener(new View.OnClickListener() {
        @Override
        public void onClick(View view) {
            HeadlessCounterFragment counterState = 
                (HeadlessCounterFragment)getFragmentManager()
                    .findFragmentByTag("counter_fragment");
            if(counterState != null) counterState.stopCounting();
        }
    });
}
{% endhighlight %}

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