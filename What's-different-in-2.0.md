RxJava 2.0 has been completely rewritten from scratch on top of the Reactive-Streams specification. The specification itself has evolved out of RxJava 1.x and provides a common baseline for reactive systems and libraries.

Because Reactive-Streams has a different architecture, it mandates changes to some well known RxJava types. This wiki page attempts to summarize what has changed and describes how to rewrite 1.x code into 2.x code.

**This page is aimed at library users, there is (going to be) a separate set of pages aimed at RxJava library developers and those who wish to develop custom operators for 2.x.**

# Maven address and base package

To allow having RxJava 1.x and RxJava 2.x side-by-side, RxJava 2.x is under the maven coordinates `io.reactivex.rxjava2:rxjava:2.x.y` and classes are accessible below `io.reactivex`.

Users switching from 1.x to 2.x have to re-organize their imports, but carefully.

# Observable and Flowable

A small regret about introducing backpressure in RxJava 0.x is that instead of having a separate base reactive class, the `Observable` itself was retrofitted. The main issue with backpressure is that many hot sources, such as UI events, can't be reasonably backpressured and cause unexpected `MissingBackpressureException` (i.e., beginners don't expect them).

We try to remedy this situation in 2.x by having `io.reactivex.Observable` non-backpressured and the new `io.reactivex.Flowable` be the backpressure-enabled base reactive class.

The good news is that operator names remain (mostly) the same. Bad news is that one should be careful when performing 'organize imports' as it may select the non-backpressured `io.reactivex.Observable` unintended.

(Remark: up for discussion.)

# Single

The 2.x `Single` reactive base type, which can emit a single `onSuccess` or `onError` has been redesigned from scratch. It's architecture now derives from the Reactive-Streams design. Its consumer type (`rx.Single.SingleSubscriber<T>`) has been changed from being a class that accepts `rx.Subscription` resources to be an interface `io.reactivex.SingleObserver<T>` that has only 3 methods:

```java
interface SingleObserver<T> {
    void onSubscribe(Disposable d);
    void onSuccess(T value);
    void onError(Throwable error);
}
```

and follows the protocol `onSubscribe (onSuccess | onError)?`.

# Completable

The `Completable` type remains largely the same. It was already designed along the Reactive-Streams style for 1.x so no user-level changes there.

Similar to the naming changes, `rx.Completable.CompletableSubscriber` has become `io.reactivex.CompletableObserver` with `onSubscribe(Disposable)`:

```java
interface CompletableObserver<T> {
    void onSubscribe(Disposable d);
    void onComplete();
    void onError(Throwable error);
}
```

and still follows the protocol `onSubscribe (onComplete | onError)?`.

# Base reactive interfaces

Following the style of extending the Reactive-Streams `Publisher<T>` in `Flowable`, the other base reactive classes now extend similar base interfaces (in package `io.reactivex`):

```java
interface ObservableSource<T> {
    void subscribe(Observer<? super T> observer);
}

interface SingleSource<T> {
    void subscribe(SingleObserver<? super T> observer);
}
```

Therefore, many operators that required some reactive base type from the user now accept `Publisher` and `XSource`:

```java
Flowable<R> flatMap(Function<? super T, ? extends Publisher<? extends R>> mapper);

Observable<R> flatMap(Function<? super T, ? extends ObservableSource<? extends R>> mapper);
```

By having `Publisher` as input this way, you can compose with other Reactive-Streams compliant libraries without the need to wrap them or convert them into `Flowable` first.

If an operator has to offer a reactive base type, however, the user will receive the full reactive class (as giving out an `XSource` is practically useless as it doesn't have operators on it):

```java
Flowable<Flowable<Integer>> windows = source.window(5);

source.compose((Flowable<T> flowable) -> 
    flowable
    .subscribeOn(Schedulers.io())
    .observeOn(AndroidSchedulers.mainThread()));
```


# Functional interfaces

Because both 1.x and 2.x is aimed at Java 6+, we can't use the Java 8 functional interfaces such as `java.util.function.Function`. Instead, we defined our own functional interfaces in 1.x and 2.x follows this tradition. 

One notable difference is that all our functional interfaces now define `throws Exception`. This is a large convenience for consumers and mappers that otherwise throw and would need `try-catch` to transform or suppress a checked exception.

```java
Flowable.just("file.txt")
.map(name -> Files.readLines(name))
.subscribe(lines -> System.out.println(lines.size()), Throwable::printStackTrace);
```

If the file doesn't exist or can't be read properly, the end consumer will print out `IOException` directly. Note also the `Files.readLines(name)` invoked without try-catch.

(Remark: up for discussion.)

## Actions

As the opportunity to reduce component count, 2.x doesn't define `Action3`-`Action9` and `ActionN` (these were unused within RxJava itself anyway). 

The remaining action interfaces were named according to the Java 8 functional types. The no argument `Action0` is replaced by the `io.reactivex.functions.Action` for the operators and `java.lang.Runnable` for the `Scheduler` methods. `Action1` has been renamed to `Consumer` and `Action2` is called `BiConsumer`. `ActionN` is replaced by the `Consumer<Object[]>` type declaration.

## Functions

We followed the naming convention of Java 8 by defining `io.reactivex.functions.Function` and `io.reactivex.functions.BiFunction`, plus renaming `Func3` - `Func9` into `Function3` - `Function9` respectively. The `FuncN` is replaced by the `Function<Object[], R>` type declaration.

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

To bridge the gap we defined abstract classes `DefaultSubscriber`, `ResourceSubscriber` and `DisposableSubscriber` (plus their `XObserver` variants) for `Flowable` (and `Observable`) respectively that offers resource tracking support (of `Disposable`s) just like `rx.Subscriber` and can be cancelled/disposed externally via `dispose()`: 

```java
ResourceSubscriber<Integer> subscriber = new ResourceSubscriber<Integer>() {
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

The original `Subscription` container types have been renamed and updated

  - `CompositeSubscription` to `CompositeDisposable`
  - `SerialSubscription` and `MultipleAssignmentSubscription` have been merged into `SerialDisposable`. The `set()` method disposes the old value and `replace()` method does not.
  - `RefCountSubscription` has been removed.

# Backpressure

The Reactive-Streams specification mandates operators supporting backpressure, specifically via the guarantee that they don't overflow their consumers when those don't request. Operators of the new `Flowable` base reactive type now consider downstream requrest amounts properly, however, this doesn't mean `MissingBackpressureException` is gone. The exception is still there but this time, the operator that can't signal more `onNext` will signal this exception instead (allowing better identification of who is not properly backpressured).

As an alternative, the 2.x `Observable` doesn't do backpressure at all and is available as a choice to switch over.

# Runtime hooks

The 2.x redesigned the `RxJavaPlugins` class which now supports changing the hooks at runtime. Tests that want to override the schedulers and the lifecycle of the base reactive types can do it on a case-by-case basis through callback functions.

The class-based `RxJavaObservableHook` and friends are now gone and `RxJavaHooks` functionality is incorporated into `RxJavaPlugins`.

# Schedulers

The 2.x API still supports the main default scheduler types: `computation`, `io`, `newThread` and `trampoline`, accessible through `io.reactivex.schedulers.Schedulers` utility class. 

The `immediate` scheduler is not present in 2.x. It was frequently misused and didn't implement the `Scheduler` specification correctly anyway; it contained blocking sleep for delayed action and didn't support recursive scheduling at all.

The `Schedulers.test()` has been removed as well to avoid the conceptional difference with the rest of the default schedulers. Those return a "global" scheduler instance whereas `test()` returned always a new instance of the `TestScheduler`. Test developers are now encouraged to simply `new TestScheduler()` in their code.

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

# Entering the reactive world

One of the design flaws of RxJava 1.x was the exposure of the `rx.Observable.create()` method that while powerful, not the typical operator you want to use to enter the reactive world. Unfortunately, so many depend on it that we couldn't remove or rename it.

Since 2.x is a fresh start, we won't make that mistake again. Each reactive base type `Flowable`, `Observable`, `Single` and `Completable` feature a safe `create` operator that does the right thing regarding backpressure (for `Flowable`) and cancellation (all):

```java
Flowable.create((FlowableEmitter<Integer> emitter) -> {
    emitter.onNext(1);
    emitter.onNext(2);
    emitter.onComplete();
}, BackpressureStrategy.BUFFER);
```

Practically, the 1.x `fromAsync` has been renamed to `Flowable.create`. The other base reactive types have similar `create` methods (minus the backpressure strategy).

# Leaving the reactive world

Apart from subscribing to the base types with their respective consumers (`Subscriber`, `Observer`, `SingleObserver` and `CompletableObserver`) and functional-interface based consumers (such as `subscribe(Consumer<T>, Consumer<Throwable>, Action)`), the formerly separate 1.x `BlockingObservable` (and similar classes for the others) has been integrated with the main reactive type. Now you can directly block for some results by invoking a `blockingX` operation directly:

```java
List<Integer> list = Flowable.range(1, 100).toList().blockingFirst();
```

(The reason for this is twofold: performance and ease of use of the library as a synchronous Java 8 Streams-like processor.)

# Operator differences

Most operators are still there in 2.x and practically all of them have the same behavior as they had in 1.x. The following subsections list each base reactive type and the difference between 1.x and 2.x.

Generally, many operators gained overloads that now allow specifying the internal buffer size or prefetch amount they should run their upstream (or inner sources).

Some operator overloads have been renamed with a postfix, such as `fromArray`, `fromIterable` etc. The reason for this is that when the library is compiled with Java 8, the javac often can't disambiguate between functional interface types.

Operators marked as `@Beta` or `@Experimental` in 1.x are promoted to standard.

## 1.x Observable to 2.x Flowable

###Factory methods:

| 1.x      | 2.x       |
|----------|-----------|
| `amb` | added `amb(ObservableSource...)` overload, 2-9 argument versions dropped |
| RxRingBuffer.SIZE | `bufferSize()` |
| `combineLatest` | added varargs overload, added overloads with `bufferSize` argument, `combineLatest(List)` dropped |
| `concat` | added overload with `prefetch` argument |
| N/A | added `concatArray` and `concatArrayDelayError` |
| N/A | added `concatArrayEager` and `concatArrayEagerDelayError` | 
| `concatDelayError` | added overloads with option to delay till the current ends or till the very end |
| `concatEagerDelayError` | added overloads with option to delay till the current ends or till the very end |
| `create(SyncOnSubscribe)` | replaced with `generate` + overloads (distinct interfaces, you can implement them all at once) |
| `create(AsnycOnSubscribe)` | not present |
| `create(OnSubscribe)` | repurposed with safe `create(FlowableOnSubscribe, BackpressureStrategy)`, raw support via `unsafeCreate()` |
| `from` | disambiguated into `fromArray`, `fromIterable`, `fromFuture` |
| N/A | added `fromPublisher` |
| `fromAsync` | renamed to `create()` |
| N/A | added `intervalRange()` |
| `limit` | dropped, use `take` |
| `merge` | added overloads with `prefetch` |
| `mergeDelayError` | added overloads with `prefetch` |
| `nest` | dropped, use manual `just` |
| `sequenceEqual` | added overload with `bufferSize` |
| `switchOnNext` | added overload with `prefetch` |
| `switchOnNextDelayError` | added overload with `prefetch` |
| `timer` | deprecated overloads dropped |
| `zip` | added overloads with `bufferSize` and `delayErrors` capabilities, disambiguated to `zipArray` and `zipIterable` |

###Instance methods:

| 1.x      | 2.x      |
|----------|----------|
| `asObservable` | renamed to `hide()`, hides all identities now |
| `buffer` | overloads with custom `Collection` supplier |
| `cache(int)` | deprecated and dropped |
| `collect(U, Action2<U, T>)` | disambiguated to `collectInto` |
| `concatMap` | added overloads with `prefetch` |
| `concatMapDelayError` | added overloads with `prefetch`, option to delay till the current ends or till the very end |
| `concatMapEager` | added overloads with `prefetch` |
| `concatMapEagerDelayError` | added overloads with `prefetch`, option to delay till the current ends or till the very end | `count` | returns `Flowable<Long>` now |
| `countLong` | dropped, use `count` |
| `distinct` | overload with custom `Collection` supplier. |
| `doOnCompleted` | renamed to `doOnComplete`, note the missing `d`! |
| `doOnUnsubscribe` | renamed to `doOnCancel` |
| N/A | added `doOnLifecylce` to handle `onSubscribe`, `request` and `cancel` peeking |
| `elementAt(Func1, int)` | dropped, use `filter(predicate).elementAt(int)` |
| `elementAtOrDefault(int, T)` | renamed to `elementAt(int, T)` |
| `elementAtOrDefault(Func1, int, T)` | dropped, use `filter(predicate).elementAt(int, T)` |
| `first(Func1)` | dropped, use `filter(predicate).first()` |
| `firstOrDefault(T)` | renamed to `first(T)` |
| `firstOrDefault(Func1, T)` | dropped, use `filter(predicate).first(T)` |
| `flatMap` | added overloads with `prefetch` |
| N/A | added `forEachWhile(Predicate<T>, [Consumer<Throwable>, [Action]])` for conditionally stopping consumption |
| `groupBy` | added overload with `bufferSize` and `delayError` option |
| `last(Func1)` | dropped, use `filter(predicate).last()` |
| `lastOrDefault(T)` | renamed to `last(T)` |
| `lastOrDefault(Func1, T)` | dropped, use `filter(predicate).last(T)` |
| `publish(Func1)` | added overload with `prefetch` |
| N/A | added `reduceWith(Callable, BiFunction)` to reduce in a Subscriber-individual manner |
| N/A | added `repeatUntil(BooleanSupplier)` |
| `repeatWhen(Func1, Scheduler)` | dropped the overload, use `subscribeOn(Scheduler).repeatWhen(Function)` instead |
| `retry` | added `retry(Predicate)`, `retry(int, Predicate)` |
| N/A | added `retryUntil(BooleanSupplier)` |
| `retryWhen(Func1, Scheduler)` | dropped the overload, use `subscribeOn(Scheduler).retryWhen(Function)` instead |
| N/A | added `sampleWith(Callable, BiFunction)` to scan in a Subscriber-individual manner |
| `single(Func1)` | dropped, use `filter(predicate).single()` |
| `singleOrDefault(T)` | renamed to `single(T)` |
| `singleOrDefault(Func1, T)` | dropped, use `filter(predicate).single(T)` |
| `skipLast` | added overloads with `bufferSize` and `delayError` options |
| `startWith` | 2-9 argument version dropped, use `startWithArray` instead |
| N/A | added `startWithArray` to disambiguate |
| `switchMap` | added overload with `prefetch` argument |
| `switchMapDelayError` | added overload with `prefetch` argument |
| N/A | added `test()` (returns TestSubscriber subscribed to this) with overloads to fluently test |
| `toBlocking().y` | inlined as `blockingY()` operators, except `toFuture` |
| N/A | added `toFuture` |
| N/A | added `toObservable` |
| `zipWith` | added overloads with `prefetch` and `delayErrors` options |







