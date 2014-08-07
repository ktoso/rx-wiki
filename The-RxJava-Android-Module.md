The `rxjava-android` module contains Android-specific bindings for RxJava. It adds a number of classes to RxJava to assist in writing reactive components in Android application.

- It provides a `Scheduler` that schedules an `Observable` on a given Android `Handler` thread, particularly the main UI thread.
- It provides operators that make it easier to deal with `Fragment` and `Activity` life-cycle callbacks.
- It provides wrappers for various Android messaging and notification components so that they can be lifted into an Rx call chain
- It provides reusable, self-contained, reactive components for common Android use cases and UI concerns. _(coming soon)_

# Binaries

You can find binaries and dependency information for Maven, Ivy, Gradle and others at [http://search.maven.org](http://search.maven.org/#search%7Cga%7C1%7Ca%3A%22rxjava-android%22).

Here is an example for [Maven](http://search.maven.org/#search%7Cga%7C1%7Ca%3A%22rxjava-android%22):

```xml
<dependency>
    <groupId>com.netflix.rxjava</groupId>
    <artifactId>rxjava-android</artifactId>
    <version>0.10.1</version>
</dependency>
```

&hellip;and for Ivy:

```xml
<dependency org="com.netflix.rxjava" name="rxjava-android" rev="0.10.1" />
```

The currently supported `minSdkVersion` is `10` (Android 2.3/Gingerbread)

# Examples

## Observing on the UI thread

You commonly deal with asynchronous tasks on Android by observing the task&#8217;s result or outcome on the main UI thread. Using vanilla Android, you would typically accomplish this with an `AsyncTask`. With RxJava you would instead declare your `Observable` to be observed on the main thread by using the `observeOn` operator:

```java
    public class ReactiveFragment extends Fragment {

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        Observable.from("one", "two", "three", "four", "five")
                .subscribeOn(Schedulers.newThread())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(/* an Observer */);
    }
```
 
This executes the Observable on a new thread, which emits results through `onNext` on the main UI thread.

## Observing on arbitrary threads
The previous example is a specialization of a more general concept: binding asynchronous communication to an Android message loop by using the `Handler` class. In order to observe an `Observable` on an arbitrary thread, create a `Handler` bound to that thread and use the `AndroidSchedulers.handlerThread` scheduler:

```java
    new Thread(new Runnable() {
        @Override
        public void run() {
            final Handler handler = new Handler(); // bound to this thread
            Observable.from("one", "two", "three", "four", "five")
                    .subscribeOn(Schedulers.newThread())
                    .observeOn(AndroidSchedulers.handlerThread(handler))
                    .subscribe(/* an Observer */)
                    
            // perform work, ...
        }
    }, "custom-thread-1").start();
```

This executes the Observable on a new thread and emits results through `onNext` on `custom-thread-1`. (This example is contrived since you could as well call `observeOn(Schedulers.currentThread())` but it illustrates the idea.)

## Fragment and Activity life-cycle

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

# See also
* [The rxjava-android readme file](https://github.com/Netflix/RxJava/tree/master/rxjava-contrib/rxjava-android)
* [Functional Reactive Programming on Android With RxJava](http://mttkay.github.io/blog/2013/08/25/functional-reactive-programming-on-android-with-rxjava/) and [Conquering concurrency - bringing the Reactive Extensions to the Android platform](https://speakerdeck.com/mttkay/conquering-concurrency-bringing-the-reactive-extensions-to-the-android-platform) by Matthias Käppler
* [Learning RxJava for Android by example](https://github.com/kaushikgopal/Android-RxJava) by Kaushik Gopal
* [Top 7 Tips for RxJava on Android](http://blog.futurice.com/top-7-tips-for-rxjava-on-android) and [Rx Architectures in Android](http://www.slideshare.net/TimoTuominen1/rxjava-architectures-on-android-8-android-livecode-32531688) by Timo Tuominen
* [FRP on Android](http://slid.es/yaroslavheriatovych/frponandroid) by Yaroslav Heriatovych
* [Rx for .NET and RxJava for Android](http://blog.futurice.com/tech-pick-of-the-week-rx-for-net-and-rxjava-for-android) by Olli Salonen
* [RxJava in Xtend for Android](http://blog.futurice.com/android-development-has-its-own-swift) by Andre Medeiros
* [RxJava and Xtend](http://mnmlst-dvlpr.blogspot.de/2014/07/rxjava-and-xtend.html) by Stefan Oehme