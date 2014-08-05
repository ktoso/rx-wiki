# Android Support

RxJava has been carefully designed to support alternative Java runtime environments and language libraries like those used on Android. Additional Android specific framework classes have been added and are currently available under [rxjava-contrib/rxjava-android](https://github.com/Netflix/RxJava/tree/master/rxjava-contrib/rxjava-android) folder, soon to be promoted into a separate RxAndroid project.

## Requirements and setup

To get started using RxJava in your Android app, add the Android module to your project dependencies. The artifacts are [available through Maven Central](http://search.maven.org/#search%7Cga%7C1%7Ca%3A%22rxjava-android%22). The coordinates for Maven and Gradle based builds are:

```
   Group ID:  com.netflix.rxjava
Artifact ID:  rxjava-android
```

The currently supported `minSdkVersion` is `10` (Android 2.3/Gingerbread)

## Getting started

You can use the core library classes to their full extend; no particular Android specific setup is required. This will likely get you a long way already. However, to fit RxJava better into existing (and somewhat conflicting) Android specific concepts such as `Looper` and friends, `Activity`, `Fragment` or `View`, framework classes were added for easier integration.

### Scheduling work on the main UI thread

Probably the most common thing you will want to do is observe an Rx sequence on the main UI thread. Android only allows you to mutate view state from the main UI thread, so any values you emit from a background thread that will update a view through a subscriber have to be rescheduled to the UI thread. Using vanilla Android, you would most likely use `AsyncTask` to do that. In RxJava, you use a `Scheduler`. We've added `HandlerThreadScheduler` and a pre-baked instance of it representing the main thread to do exactly that:

```
Observable.from("Hello", "World", "!")
    .subscribeOn(Schedulers.newThread()) // strings are emitted on background thread
    .observeOn(AndroidSchedulers.mainThread()) // strings are observed on UI thread
    .subscribe((value) -> {
         textView.setText(value); // this is now safe to do!
    });
```

Transforming a sequence using `observeOn(AndroidSchedulers.mainThread())` will make sure that any Rx notifications emitted from the sequence will be passed through Android's main message looper, so that any subscriber receives its callbacks (`onNext`, `onError`, `onCompleted`) on the main UI thread. This makes it safe to access and mutate views in a subscriber or chained operator.

### Fragment and Activity life-cycle

One thing that's tricky to deal with on Android is running asynchronous actions that access framework objects in their callbacks. That's because Android may decide to destroy an Activity, for instance, while a background thread is still running. An attempt will be made to access views on the now dead Activity, resulting in a crash. (It will also create a memory leak, since your background thread holds on to the Activity even though it's not visible anymore.)

This is no different when using RxJava on Android, but one can deal with the problem in a more elegant way through the use of `Subscription`s, and a number of operators. In general, when running an `Observable` inside an `Activity` which subscribes to the result (either directly or through an inner class), you must ensure to unsubscribe from the sequence in `onDestroy`:

```
// MyActivity
private Subscription subscription;

protected void onCreate(Bundle savedInstanceState) {
    this.subscription = observable.subscribe(this);
}

...

protected void onDestroy() {
    this.subscription.unsubscribe();
    super.onDestroy();
}
```

This will ensure that all references to the subscriber (the Activity) will be released as soon as possible, and no more notifications will arrive at the subscriber through `onNext`.

One problem with this is that if the Activity is destroyed because of a change in screen orientation, the observable will fire again in `onCreate`. This can be prevented by using the `cache` or `replay` operators, while making sure the Observable somehow survives the Activity life-cycle (e.g. by storing it in a global cache, in a Fragment, etc.) Using either operator will make sure that when subscribing to an Observable that's already "running", values received during the time of detachment from the Activity will be "played back", and in-flight notifications will be delivered as usual.

