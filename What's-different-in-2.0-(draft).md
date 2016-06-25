RxJava 2.0 has been completely rewritten from scratch on top of the Reactive-Streams specification. The specification itself has evolved out of RxJava 1.x and provides a common baseline for reactive systems and libraries.

Because Reactive-Streams has a different architecture, it mandates changes to some well known RxJava types. This wiki page attempts to summarize what has changed and describes how to rewrite 1.x code into 2.x code.

**This page is aimed at library users, there is (going to be) a separate set of pages aimed at RxJava library developers and those who wish to develop custom operators for 2.x.**

# Maven address and base package

To allow having RxJava 1.x and RxJava 2.x side-by-side, RxJava 2.x is under the maven coordinates `io.reactivex.rxjava:rxjava:2.x.y` and classes are accessible below `io.reactivex`.

Users switching from 1.x to 2.x have to re-organize their imports, but carefully.

# Observable and Flowable

A small regret about introducing backpressure in RxJava 0.x is that instead of having a separate base reactive class, the `Observable` itself was retrofitted. The main issue with backpressure is that many hot sources, such as UI events, can't be reasonably backpressured and cause unexpected `MissingBackpressureException` (i.e., beginners don't expect them).

We try to remedy this situation in 2.x by having `io.reactivex.Observable` non-backpressured and the new `io.reactivex.Flowable` be the backpressure-enabled base reactive class.

The good news is that operator names remain (mostly) the same. Bad news is that one should be careful when performing 'organize imports' as it may select the non-backpressured `io.reactivex.Observable` unintended.

(Remark: up for discussion.)

# Functional interfaces

Because both 1.x and 2.x is aimed at Java 6+, we can't use the Java 8 functional interfaces such as `java.util.function.Function`. Instead, we defined our own functional interfaces in 1.x and 2.x follows this tradition.

(Remark: up for discussion.)

## Actions

As the opportunity to reduce component count, 2.x doesn't define `Action3`-`Action9` and `ActionN` (these were unused within RxJava itself anyway). 

The remaining action interfaces were named according to the Java 8 functional types. The no argument `Action0` is replaced by the `java.lang.Runnable` that allows more native interoperation with Java features. `Action1` has been renamed to `Consumer` and `Action2` is called `BiConsumer`. `ActionN` is replaced by the `Action1<Object[]>` type declaration.

## Functions

We followed the naming convention of Java 8 by defining `io.reactivex.functions.Function` and `io.reactivex.functions.BiFunction`, plus renaming `Func3` - `Func9` into `Function3` - `Function9` respectively. The `FuncN` is replaced by the `FuncN<Object[], R>` type declaration.

In addition, operators requiring a predicate no longer use `Func1<T, Boolean>` but have a separate, primitive-returning type of `Predicate<T>` (allows better inlining due to no autoboxing).

# Subscriber

The Reactive-Streams specification has its own Subscriber as an interface. This interface is lightweight and combines request management with cancellation into a single interface `org.reactivestreams.Subscription` instead of having `rx.Producer` and `rx.Subscription` separately. This allows creating stream consumers with less internal state than the quite heavy `rx.Subscriber` of 1.x.

```java
Flowable.range(1, 10).subscribe(new Subscriber<Integer>() {
    @Override
    public void onSubscribe(Subscription s) {
        s.request(Long.MAX_VALUE);
    }

    @Override
    public void onNext(Integer t) {
        System.out.println(t);
    }

    @Override
    public void onError(Throwable t) {
        t.printStackTrace();
    }

    @Override
    public void onComplete() {
        System.out.println("Done");
    }
});
```

Due to the name conflict, replacing the package from `rx` to `org.reactivestreams` is not enough. In addition, `org.reactivestreams.Subscriber` has no notion for adding resources to it, cancelling it or requesting from the outside.

To bridge the gap we defined abstract classes `AsyncSubscriber` and `AsyncObserver` for `Flowable` and `Observable` respectively that offers resource tracking support (of `Disposable`s) just like `rx.Subscriber` and can be cancelled/disposed externally via `dispose()`: 

```java
AsyncSubscriber<Integer> subscriber = new AsyncSubscriber<Integer>() {
    @Override
    public void onStart() {
        request(Long.MAX_VALUE);
    }

    @Override
    public void onNext(Integer t) {
        System.out.println(t);
    }

    @Override
    public void onError(Throwable t) {
        t.printStackTrace();
    }

    @Override
    public void onComplete() {
        System.out.println("Done");
    }
};

Flowable.range(1, 10).delay(1, TimeUnit.SECONDS).subscribe(subscriber);

subscriber.dispose();
```

Note also that due to Reactive-Streams compatibility, the method `onCompleted` has been renamed to `onComplete` without the trailing `d`.

# Subscription

In RxJava 1.x, the interface `rx.Subscription` was responsible for stream and resource lifecycle management, namely unsubscribing a sequence and releasing general resources such as scheduled tasks. The Reactive-Streams specification took this name for specifying an interaction point between a source and a consumer: `org.reactivestreams.Subscription` allows requesting a positive amount from the upstream and allows cancelling the sequence.

To avoid the name clash, the 1.x `rx.Subscription` has been renamed into `io.reactivex.Disposable` (somewhat resembling .NET's own IDisposable).

Because Reactive-Streams base interface, `org.reactivestreams.Publisher` defines the `subscribe()` method as `void`, `Flowable.subscribe(Subscriber)` no longer returns any `Subscription` (or `Disposable`). The other base reactive types also follow this signature with their respective subscriber types.

The other overloads of `subscribe` now return `Disposable` in 2.x.

# Backpressure

The Reactive-Streams specification mandates operators supporting backpressure, specifically via the guarantee that they don't overflow their consumers when those don't request. Operators of the new `Flowable` base reactive type now consider downstream requrest amounts properly, however, this doesn't mean `MissingBackpressureException` is gone. The exception is still there but this time, the operator that can't signal more `onNext` will signal this exception instead (allowing better identification of who is not properly backpressured).

As an alternative, the 2.x `Observable` doesn't do backpressure at all and is available as a choice to switch over.

# Runtime hooks

The 2.x redesigned the `RxJavaPlugins` class which now supports changing the hooks at runtime. Tests that want to override the schedulers and the lifecycle of the base reactive types can do it on a case-by-case basis through callback functions.

# Schedulers

The 2.x API still supports the main default scheduler types: `computation`, `io`, `newThread`, `trampoline` and `test`, accessible through `io.reactivex.schedulers.Schedulers` utility class. Due to the blocking-sleep in `immediate` there is a plan to drop it entirely from 2.x.

The `io.reactivex.Scheduler` abstract base class now supports scheduling tasks directly without the need to create and then destroy a `Worker` (which is often forgotten):

```java
public abstract class Scheduler {
    
    public Disposable scheduleDirect(Runnable task) { ... }
 
    public Disposable scheduleDirect(Runnable task, long delay, TimeUnit unit) { ... }

    public Disposable scheduleDirectPeriodically(Runnable task, long initialDelay, 
        long period, TimeUnit unit) { ... }

    public long now(TimeUnit unit) { ... }

    // ... rest is the same: lifecycle methods, worker creation
}
```

The main purpose is to avoid the tracking overhead of the `Worker`s for typically one-shot tasks. The methods have a default implementation that reuses `createWorker` properly but can be overridden with more efficient implementations if necessary.

The method that returns the scheduler's own notion of current time, `now()` has been changed to accept a `TimeUnit` to indicate the unit of measure.