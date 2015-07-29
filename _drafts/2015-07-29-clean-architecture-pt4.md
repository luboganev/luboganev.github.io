---
layout: post
title: Android clean architecture part 4
description: "This post presents a demo application built with the VIPER architecture in mind. It defines a common set of naming rules and patterns to be followed, when developing an Android application with the clean architecture in mind."
modified: 2015-07-29
category: draft
tags: [android, MVP, dependecy injection, VIPER, Dagger, tip, pattern, architecture, software development]
share: true
---

Although one could write a detailed description of the concepts and ideas behind an architecture, I easiest way to understand something is to see an example. Therefore, I have built a simple demo application, which is built with the VIPER architecture in mind. The following sections describe the application and some specifics, related to Android.

### Introduction to the demo
The demo app is called Car brands and is an open-source project hosted at GitHub. You can find the repository here: [Repository Link](https://github.com/luboganev/Carbrands)

If you do not feel like building the app yourself or cloning the repository, you can alternatively download the source and a ready built debug apk file from the release section of the project at GitHub. Please find it here: [Release Link](https://github.com/luboganev/Carbrands/releases/tag/v1.1)

### Overview of the app architecture with Dagger

The application has an extremely simple List/Detail structure. The following screenshot shows how the application packages look like in Android Studio.

![Demo project structure]({{ site.url }}/images/2015-07-29-clean-architecture-pt4/carbrands_project_structure.png)

It has two simple Activities, which represent the two screens of the application. Thus, there are two VIPER modules - *carbrands*, which contains components related the list, and *carbrandDetail*, which contains components related to the detail. All instantiating of VIPER components such as Interactors and Presenters and their dependencies is done inside the Dagger Modules classes. There are in total four different Dagger Modules, which are depicted in the following diagram.

![Dagger modules structure]({{ site.url }}/images/2015-07-29-clean-architecture-pt4/dagger_modules_structure.png)

The root Dagger Object Graph contains the main AppModule. It provides instances of the mock implementations of a data store and location manager to all classes which might need them. The NavigatorModule provides instance of the Navigator object which contains logic for navigating from the ListActivity to the DetailActivity. The CarBrandsListModule provides instances of the Presenter and Interactor related to the list VIPER module. The CarBrandDetailModule provides instances of the Presenter and Interactor related to the detail VIPER module.

Since the UI of the application has a shorter lifecycle in comparison with the application process, we do not need to persist in memory all possible dependencies related to the list and detail VIPER modules all the time. Therefore, we use the *scoped graph* Dagger functionality and we build an extended version of the Object Graph, containing the main Application Object Graph and the Modules related only to the Activity being shown. After the Activity goes away, its scoped object graph with all its instances is garbage collected, without affecting the instances of dependencies living in the Application object graph.

In the implementation, almost dependency injection is performed through constructor injection. This all happens in the Dagger Module classes, when instantiating the concrete implementations of the Interactors and Presenters. The only object we cannot create are the Android Activities. Thus, we need to make field injection there, after these objects have been instantiated by the Android framework. In order to encapsulate the functionality related to scoping graphs and field-injecting an instantiated Activity, the demo app has a BaseDaggerActivity abstract class, which performs this commons tasks. It uses the custom implementation of the Application class, which contains the logic for creating the initial root Application Dagger Object Graph. The following code snippets depict what are possibly the two most important classes in the demo application.

{% highlight java %}
public abstract class BaseDaggerActivity extends AppCompatActivity {
    private @Nullable ObjectGraph mActivityGraph;

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        List<Object> modules = getActivityModules();
        if(modules != null && modules.size() > 0) {
            mActivityGraph = CarBrandsApplication.get(this).createScopedGraph(modules.toArray());
        }
        if(shouldInjectSelf()) {
            if (mActivityGraph != null) {
                mActivityGraph.inject(this);
            } else {
                CarBrandsApplication.get(this).inject(this);
            }

            // Signal to Activities extending the BaseActivity, that the injection is completed
            onInjected(savedInstanceState);
        }
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        mActivityGraph = null;
    }

    /**
     *  This method should return a list of modules to be added as a scoped object graph
     *  to the Application's object graph. If it does not provide any modules,
     *  the current Activity does not generate its own scoped object graph and uses the
     *  Application's object graph instead.
     */
    protected @Nullable List<Object> getActivityModules() { return null; }

    /**
     *  This method returns whether the current Activity itself
     *  should be injected with dependencies
     */
    protected boolean shouldInjectSelf() { return false; }

    /**
     *  A helper method used to inject objects. It uses the Activity's the scoped object
     *  graph if one exists. In case the Activity does not have its own scoped object graph,
     *  the Application's object graph is used for the injection.
     *
     * @param object
     *      The object to be injected with dependencies
     */
    protected void inject(@NonNull Object object) {
        if(mActivityGraph == null) {
            CarBrandsApplication.get(this).inject(object);
        } else {
            mActivityGraph.inject(object);
        }
    }

    /**
     * This method gets called once the Activity has been injected.
     * It is a hook you can use if you want to do something as early as
     * in the onCreate() but still need the injected dependencies
     *
     * @param savedInstanceState
     *      The reference to the savedInstanceState from the onCreate method.
     */
    protected void onInjected(@Nullable Bundle savedInstanceState) {}
}
{% endhighlight %}

{% highlight java %}
public class CarBrandsApplication extends Application {
    private ObjectGraph objectGraph;

    @Override
    public void onCreate() {
        super.onCreate();
        objectGraph = ObjectGraph.create(new AppModule(this));
    }

    /**
     * A helper method, which creates a scoped Dagger object graph, by adding the input
     * modules to the root Application object graph.
     *
     * @param modules
     *      The additional modules which should be included in the scoped object graph.
     * @return
     *      An instance of the created scoped object graph
     */
    public ObjectGraph createScopedGraph(Object... modules) {
        return objectGraph.plus(modules);
    }

    /**
     *  A helper method which injects objects with their dependencies from the
     *  the Application's object graph
     *
     * @param object
     *      The object to be injected
     */
    public void inject(@NonNull Object object) {
        objectGraph.inject(object);
    }

    /**
     *  A helper method which gets a reference to the Application from an input Activity instance
     */
    public static CarBrandsApplication get(@NonNull Activity activity) {
        return (CarBrandsApplication) activity.getApplication();
    }
{% endhighlight %}

For even clearer documentation and comments inside these two classes, please refer to the code of the demo app.

### Specifics of a VIPER module in this context

### Saving state

### Long-running atomic operations