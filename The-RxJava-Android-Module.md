The `rxjava-android` module contains Android-specific bindings for RxJava. It adds a number of classes to RxJava to assist in writing reactive components in Android application.

- It provides a `Scheduler` that schedules an `Observable` on a given Android `Handler` thread, particularly the main UI thread.
- It provides base `Observer` implementations that make guarantees concerning reliable and thread-safe use throughout `Fragment` and `Activity` life-cycle callbacks. _(coming soon)_
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