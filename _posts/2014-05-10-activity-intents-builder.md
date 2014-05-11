---
layout: post
title: Activity Intents builder
description: "This post proposes a clean pattern for building custom Intents needed to start explicitly an activity"
modified: 2014-05-11
category: blog
tags: [android, tip, pattern, software development, builder, activity, intent]
share: true
---

This post proposes an Android development pattern I have came up with while trying to refactor an ugly looking code (I was responsible for most of it anyway). You probably are familiar with programming a simple piece of code, which at some point after adding a ton of new options starts to get seriously cumbersome. At this point one has to do some refactoring to eliminate at least the serious code duplication which inevitably already exists.

### Where did the initial idea for the pattern come from?

Probably all of you are familiar with the proposed pattern for creating Fragments in Android. The official documentation recommends that instances of fragments should not be  created via a constructor, but instead by implementing a pattern including a static _newInstance()_ function. By doing this, all necessary fragment arguments names and types could be hidden from the outside world, which leads to better encapsulation of the fragment class itself. If you are not familiar with this pattern please find it in the official [Fragment](http://developer.android.com/reference/android/app/Fragment.html) documentation. 

### Fragments' arguments and intents' extras

Similarly to Fragments, an Activity also could receive "Arguments" when it is being launched. All activities in Android are being launched through Intents, and Intents can contain a Bundle with extras. The difference is that the methods for working with these Bundles have different names. The more important difference is of course the fact that Fragments contain a Bundle with arguments, while the Bundle with Extras is not part of the Activity class itself, but is contained in its launching Intent. However, the extras and arguments have the same nature and could be treated in the same way.

### The _getLaunchingIntent()_ pattern

As we already discussed, the launching Intent's Bundle with extras is similar to the Bundle with arguments which the Fragments have. Therefore, one could also use the _newInstance()_ pattern when instantiating the Activities' launching Intent. A more appropriate method name in this case would be something like  _getLaunchingIntent()_. The following code snippet illustrates this pattern with a simple activity which uses a custom title sent to it through the launching Intent.

{% highlight java %}
public class AwsumActivity extends Activity {

    private static String INTENT_EXTRA_TITLE = "title";

    public static Intent getLaunchingIntent(Context ctx, String title) {
            Intent i = new Intent(ctx, AwsumActivity.class);
        i.putExtra(INTENT_EXTRA_TITLE, title);
        return i;
    }

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_awsum);

        if(getIntent().hasExtra(INTENT_EXTRA_TITLE)) {
            setTitle(getIntent().getStringExtra(INTENT_EXTRA_TITLE));
        }
    }
}
{% endhighlight %}

I have initially used this approach and it was fine at the beginning as there were not that many parameters that had to be sent to the Activity. However it got problematic when I had to sent more than a couple of extras in different combinations. The following snippet show the explosion of methods which is needed in order to cover all possible combinations for just two extras:

{% highlight java %}
private static String INTENT_EXTRA_TITLE = "title";
private static String INTENT_EXTRA_PAGE_NUMBER = "page_number";

public static Intent getLaunchingIntent(Context ctx, String title) {
    Intent i = new Intent(ctx, AwsumActivity.class);
    i.putExtra(INTENT_EXTRA_TITLE, title);
    return i;
}

public static Intent getLaunchingIntent(Context ctx, int pageNumber) {
    Intent i = new Intent(ctx, AwsumActivity.class);
    i.putExtra(INTENT_EXTRA_PAGE_NUMBER, pageNumber);
    return i;
}

public static Intent getLaunchingIntent(Context ctx, String title, int pageNumber) {
    Intent i = new Intent(ctx, AwsumActivity.class);
    i.putExtra(INTENT_EXTRA_TITLE, title);
    i.putExtra(INTENT_EXTRA_PAGE_NUMBER, pageNumber);
    return i;
}
{% endhighlight %}

Apart from the obvious code duplication, it could get even worse if different parameters share the same type. In such case we cannot overload the method for the different parameters, because they have the same type and the compiler has no idea which method would we like to call afterwards. Then we need to use different method names which would make the pattern very cumbersome to use and would eventually defy the original purpose - clean code and nice encapsulation.

### Implementing an Intent builder

The solution to this problem is to use the [Builder object creational pattern](http://sourcemaking.com/design_patterns/builder). The following code snipper demonstrates an implementation of this pattern which additionally includes a nice chaining functionality and bit flags, both of which I personally love to use.

{% highlight java %}
private static String INTENT_EXTRA_TITLE = "title";
private static String INTENT_EXTRA_PAGE_NUMBER = "page_number";

public static class AwsumIntentBuilder {
    private String mTitle;
    private int mPageNumber;

    // Bit flags which define which parameters should be added as extras to the Intent
    private int mWithParameterFlags;
    private static final int FLAG_WITH_TITLE = 1;
    private static final int FLAG_WITH_PAGE_NUMBER = 2;

    private AwsumIntentBuilder() {
        mWithParameterFlags = 0; // By default no extras will be included
    }

    public static AwsumIntentBuilder getBuilder() {
        AwsumIntentBuilder builder = new AwsumIntentBuilder();
        return builder;
    }

    public AwsumIntentBuilder withTitle(String title) {
        mWithParameterFlags = mWithParameterFlags | FLAG_WITH_TITLE;
        mTitle = title;
        return this;
    }

    public AwsumIntentBuilder withPageNumber(int pageNumber) {
        mWithParameterFlags = mWithParameterFlags | FLAG_WITH_PAGE_NUMBER;
        mPageNumber = pageNumber;
        return this;
    }

    public Intent build(Context ctx) {
        Intent i = new Intent(ctx, AwsumActivity.class);

        if((mWithParameterFlags & FLAG_WITH_TITLE) != 0) 
            i.putExtra(INTENT_EXTRA_TITLE, mTitle);
        if((mWithParameterFlags & FLAG_WITH_PAGE_NUMBER) != 0) 
            i.putExtra(INTENT_EXTRA_PAGE_NUMBER, mPageNumber);

        return i;
    }
}

.....

// Sample usage when we need to start the AwsumActivity from another Activity
Intent i = AwsumActivity.AwsumIntentBuilder.getBuilder()
                    .withTitle("Awsum title")
                    .withPageNumber(10)
                    .build(this)
startActivity(i);
{% endhighlight %}

To sum up, this pattern could be also applied to the case of Fragments, when we need to instantiate a Fragment with a complex combination of arguments. It just needs some small tweaks so that the _build()_ method does not build an instance of an Intent, but instead builds an instance of the needed Fragment and sets its arguments.

### Update (11.05.2014)

After reviewing my code, I have come to the conclusion that it is just too general and complex. It implements the pattern in a way, which does not take any advantage of the Bundle class properties until the very last moment when the Intent is being built. The following example illustrates an optimized version of the builder, which has just one member variable and does not need to do any fancy bitmask operations.

{% highlight java %}
private static String INTENT_EXTRA_TITLE = "title";
private static String INTENT_EXTRA_PAGE_NUMBER = "page_number";

public static class AwsumIntentBuilder {
    private Bundle mExtras;

    private AwsumIntentBuilder() {
        mExtras = new Bundle();
    }

    public static AwsumIntentBuilder getBuilder() {
        AwsumIntentBuilder builder = new AwsumIntentBuilder();
        return builder;
    }

    public AwsumIntentBuilder withTitle(String title) {
        mExtras.putString(INTENT_EXTRA_TITLE, title);
        return this;
    }

    public AwsumIntentBuilder withPageNumber(int pageNumber) {
        mExtras.putInt(INTENT_EXTRA_PAGE_NUMBER, pageNumber);
        return this;
    }

    public Intent build(Context ctx) {
        Intent i = new Intent(ctx, AwsumActivity.class);
        i.putExtras(mExtras);
        return i;
    }
}
    
.....

// Sample usage when we need to start the AwsumActivity from another Activity
Intent i = AwsumActivity.AwsumIntentBuilder.getBuilder()
                    .withTitle("Awsum title")
                    .withPageNumber(10)
                    .build(this)
startActivity(i);
{% endhighlight %}