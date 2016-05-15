---
layout: post
title: Android clean architecture part 5
description: "This post builds upon the architecture presented in the other posts of this series and introduces significant improvements in some architecture concepts and their sample implementation"
modified: 2016-05-15
category: post
tags: [android, MVP, dependecy injection, VIPER, Dagger, tip, pattern, architecture, software development]
share: true
---

Some time has passed since the last clean architecture post. During this time the described architecture implementation has shown some of its drawbacks. In this post I will shortly describe the problems with the current implementation and propose some solutions to these problems. As usual you can find the updated version of the demo application Carbrands in its repository at GitHub - [Repository Link](https://github.com/luboganev/Carbrands).

### So what are the problems?
All the problems with the proposed implementation actually originate from a single flawed architectural decision - storing scoped object graphs as field in their corresponding _Activity_. As a result, each Dagger scoped object graph gets destroyed and recreated together with the _Activity_ each time a UI configuration change occurs. This leads to the folowing drawbacks of the architecture:

* Potentially a lot of object instantiations due to creating a new scoped graph and all its objects on each configuration change
* _Presenter_ and _Interactor_ implementations have to rely on the framework's classes like _Bundle_ and its lifecycle event callbacks in order to save and restore their state.
* Having a _Presenter_ and _Interactor_ tightly coupled to the UI lifecycle makes them very hard to test due to tight dependencies to framework classes
* Long-running operations which need to be atomic cannot be implemented at all inside these modules due to their instability

The following paragraphs provide some more detail for each of the problems above.

#### Too many object allocations
Well, that's probably a no brainer. Recreating _Presenter_ and _Interactor_ instances and all their dependencies on each configuration change is not cheap. This takes some time and might have a negative effect on the performance of the app. In addition, the extra memory allocations means more work for the garbage collector, which also leads to sloppy UI performance.

#### Presenters, Interactors and Bundles. Oh my!
It might not look like a bad idea at first, but it proved to be really bad in the long term. You see, having to save and restore the state of _Presenter_ and _Interactor_ implementations in _Bundle_ increases their complexity with boilerplate code. The initial idea of the clean architecture is to be, well, clean. So pushing bundles and lifecycle methods as deep as an _Interactor_ is contributing to neither cleaner nor simpler code. 

In addition, it can also be error-prone. Since _Bundle_ works with key-value pairs, what happens if two components want to use the same key for different things to store? In an example scenario this will lead to an _Interactor_ overwriting value relate to the state of a Presenter or some value related to the UI, thus producing errors or even crashes due to wrong types. In order to prevent this, one has to come up with some convention that guarantees unique _Bundle_ keys. This further increases the complexity and introduces even more boilerplate code.

#### Testing and framework classes and lifecycle
Saving and restoring state of _Presenter_ and _Interactor_ through _Bundle_ and lifecycle callback methods makes testing and especially Unit testing very hard. Having references to framework classes instead of only pure Java inside these components means that you have to use libraries like _Roboelectric_ in order to be able to run Unit tests without an emulator. Don't get me wrong, Roboelectric is great, but in this case it solves a problem, which would not exist at all in a better architecture.

But let's say the only framework class we have is _Bundle_, we can mock it easily with something like _Mockito_, right? The class itself yes, but which values should it return? Which are relevant to the tested component? It means that you have to go through the code of the whole component and check what keys and values it expects to get from the _Bundle_. In other words, such components' dependencies are not clearly visible from their public API, which makes writing good tests hard, because the test has to know too much.  

#### Long-running atomic operations
Let's say that an _Interactor_ needs to perform a network request, which does somethings important like performing a payment. If a configuration change happens while the request is running, we end up with a brand new _Interactor_ which only knows that a request was sent. But it has no idea if it has reached the server, if it has failed or succeeded. It will also never get back the result of this operation coming from the server. 

This problem was already discussed into the previous post, which suggests that such _Interactor_ should live in the Application object graph and not in any of the scoped object graphs. This, however, means that once allocated this Interactor will stay in memory as long as the process lives, which is not that efficient. In addition, scoped graph Interactors have to somehow communicate with this special Interactor, which introduces yet more complexity.

### How can we do better?
Before we start looking in some aspects of the improved implementation, here's a link to the actual repository version this post refers to: [Repository Link for v2.0](https://github.com/luboganev/Carbrands/releases/tag/v2.0).

In order to fix the problems described above we need to tackle the main flaw of the current architecture, namely the fact that the instances of the scoped object graphs are stored inside the _Activity_. So how can we eliminate this? 

#### The ObjectGraphHolder
It's actually pretty simple - we just need to move them to a component which lives longer than the UI. This component is the _ObjectGraphHolder_ - a singleton which has a map containing all scoped object graphs as well as the root object graph. It provides a couple of helpful methods which allow for putting, getting and removing _ObjectGraph_ instances from it. It also contains a helper method for creating a scoped object graph by adding modules to an existing one.

{% highlight java %}

private Map<Long, ObjectGraph> objectGraphMap;
private long objectGraphIdAutoIncrement;

public @Nullable ObjectGraph getObjectGraph(long objectGraphId) {
    return objectGraphMap.get(objectGraphId);
}

public long putObjectGraph(@NonNull ObjectGraph objectGraph) {
    objectGraphMap.put(objectGraphIdAutoIncrement, objectGraph);
    return objectGraphIdAutoIncrement++;
}

public void removeObjectGraph(long objectGraphId) {
    objectGraphMap.remove(objectGraphId);
}

public static @NonNull ObjectGraph createScopedGraph(@NonNull ObjectGraph parentObjectGraph,
                              @Nullable Object... modules) {
    if (modules != null && modules.length > 0) {
        return parentObjectGraph.plus(modules);
    }

    return parentObjectGraph;
}
{% endhighlight %}

This implementation maps instances of the _ObjectGraph_ to unique auto-incrementing ids. You might wonder, why not simply use the the _Activity.class_ as a key? You definitely can, but this turns out to be a bad idea, when multiple instances of the same Activity can appear in one or multiple tasks. 

Please notice the _removeObjectGraph(id)_ method, which is used to remove object graphs related to an _Activity_ that is finished. In the previous implementation the scoped _ObjectGraph_ was garbage collected each time an _Activity_ was destroyed, but now we have to take care of this ourselves and call this remove method when an _Activity_ is finishing and will not come back.

#### The new BaseDaggerActivity
The second important component we need to take in account is the _BaseDaggerActivity_. It does not contain instance of its scoped ObjectGraph anymore, but calls the methods of the _ObjectGraphHolder_. The most important functionality can be found in the snipper below

{% highlight java %}

private long objectGraphId = -1;
private static final String STATE_EXTRA_OBJECT_GRAPH_ID = "objectgraph_id";

@Override
protected void onCreate(@Nullable Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);

    // Check for stored id and restore it
    if (savedInstanceState != null) {
        objectGraphId = savedInstanceState.getLong(STATE_EXTRA_OBJECT_GRAPH_ID, -1);
    }

    // Check if an object graph already exists for this activity
    ObjectGraph activityObjectGraph = ObjectGraphsHolder.getInstance().getObjectGraph(objectGraphId);

    if (activityObjectGraph == null) {
        // Create a new object graph, which adds to the root application object graph
        ObjectGraph rootObjectGraph = ObjectGraphsHolder.getInstance().getApplicationObjectGraph();
        activityObjectGraph = ObjectGraphsHolder.createScopedGraph(rootObjectGraph, getActivityModules());
        objectGraphId = ObjectGraphsHolder.getInstance().putObjectGraph(activityObjectGraph);
    }

    // Inject dependencies
    activityObjectGraph.inject(this);

    ....
}

@Override
protected void onSaveInstanceState(Bundle outState) {
    super.onSaveInstanceState(outState);

    // Store the current object graph id to survive configuration change
    outState.putLong(STATE_EXTRA_OBJECT_GRAPH_ID, objectGraphId);
}

@Override
protected void onDestroy() {
    super.onDestroy();

    // Any other call to this method is due to configuration change or low memory.
    // We want to release the stored object graph only when the activity is truly
    // finishing.
    if (isFinishing()) {
        onDestroyObjectGraph();
    }
}

protected void onDestroyObjectGraph() {
    // Remove the current object graph from the holder
    ObjectGraphsHolder.getInstance().removeObjectGraph(objectGraphId);
}
{% endhighlight %}

Each time the _OnCreate()_ method is invoked, we check if the _ObjectGraphHolder_ already contains an _ObjectGraph_ with the particular id related to this _Activity_. If it doesn't we, create a new scoped graph and put it in the _ObjectGraphHolder_, which gives us in return a new id for it. 

Since the id, which the _ObjectGraphHolder_ gives us when we put a new _ObjectGraph_ in it, is our only way of accessing it, we need to save it through configuration changes. When a configuration change occurs, we first try to restore it and after that we are be able to request the stored _ObjectGraph_ again in the _OnCreate()_ method.

The final piece of the _ObjectGraph_ handling is making sure to remove it from the _ObjectGraphHolder_ when its related _Activity_ finishes. This is performed in the _onDestroyObjectGraph()_ which is called from the _onDestroy()_ method only if the _Activity_ is finishing.

#### The new Navigator
The main purpose of the _Navigator_ class is to provide an abstraction around the navigation methods of an _Activity_ such as _startActivity()_ as well as the building of _Intent_ objects. In order to be able to do that, it needs to have an instance to the currently visible _Activity_. This becomes problematic when the instance of the _Navigator_ is retained through configuration change, because the Activity it contains gets destroyed. Therefore its implementation has been updated a bit and now it gets the latest _Activity_ through a setter.

{% highlight java %}
private Activity mActivity;
public void setActivity(Activity activity) {
    mActivity = activity;
}
{% endhighlight %}

This setter is being invoked in the new _BaseDaggerActivity_, which makes sure it keeps the instance inside the Navigator always fresh. This way, we also prevent leaking a destroyed _Activity_, because the reference inside the _Navigator_ points always to the newest instance of the currently visible _Activity_. The following code snippet of _BaseDaggerActivity_ shows how this is performed.

{% highlight java %}
@Inject Navigator navigator;

@Override
protected void onCreate(@Nullable Bundle savedInstanceState) {
    ....

    // Make sure to update the navigator's current activity
    navigator.setActivity(this);
    onInjected(savedInstanceState);
}

@Override
protected void onStart() {
    super.onStart();
    // Make sure to update the navigator's current activity
    navigator.setActivity(this);
}

@Override
protected void onResume() {
    super.onResume();
    // Make sure to update the navigator's current activity
    navigator.setActivity(this);
}
{% endhighlight %}

#### The new Presenter and Interactor lifecycle
Having the whole scoped object graph survive configuration changes means that we have basically no lifecycle at all. _Presenter_ and _Interactor_ instances simply get instantiated through their constructor and then are garbage collected when the scoped _ObjectGraph_ is removed from the _ObjectGraphHolder_.

However, reference to the Views cannot be injected through the constructor of a _Presenter_ anymore. This is due to the fact that when a cofiguration change occurs, we have a new _View_ instance and we have to update the reference to it inside a _Presenter_ with this new instance. The following snippets show how this is performed in the _CarBrandsActivity_ and _CarBrandsPresenter_.

{% highlight java %}
public class CarBrandsPresenter implements CarBrandsPresenterInput, CarBrandsInteractorOutput {
    private CarBrandsPresenterOutput mView;
    
    ....
    
    @Override
    public void setView(@NonNull CarBrandsPresenterOutput view) {
        mView = view;
    }
{% endhighlight %}

{% highlight java %}
public class CarBrandsListActivity extends BaseDaggerActivity implements CarBrandsPresenterOutput 
....
{

    @Inject CarBrandsPresenterInput presenter;
    
    @Override
    protected void onInjected(Bundle savedInstanceState) {
        super.onInjected(savedInstanceState);
        presenter.setView(this);
        
        ....
    }
{% endhighlight %}

Finally, sometimes we want to get notified when the whole VIPER module is no longer needed, so that we can for example abort any long-running operations inside an _Interactor_. We can then override the _onDestroyObjectGraph()_ method in our _Activity_ implementation and notify the _Presenter_ and _Interactor_ accordingly. Doing this allows these components to do any required cleaning of resources and should be always performed, so that no memory leaks can occur.

### Summary
The new architecture implementation described in this post have evolved over the last several months. The needed changes have been performed in a couple of steps and have proven to be significant improvement over the existing implementation in terms of code simplicity, testing, flexibility and separation of concerns. It is however far from perfect and can always be improved further.