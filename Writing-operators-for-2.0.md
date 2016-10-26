# Introduction

Writing operators, source-like (`fromAsync`) or intermediate-like (`flatMap`) **has always been a hard task to do in RxJava**. There are many rules to obey, many cases to consider but at the same time, many (legal) shortcuts to take to build a well performing code. Now writing an operator specifically for 2.x is 10 times harder than for 1.x. If you want to exploit all the advanced, 4th generation features, that's even 2-3 times harder on top (so 30 times harder in total).

*(If you have been following [my blog](http://akarnokd.blogspot.hu/) about RxJava internals, writing operators is maybe only 2 times harder than 1.x; some things have moved around, some tools popped up while others have been dropped but there is a relatively straight mapping from 1.x concepts and approaches to 2.x concepts and approaches.)*

In this article, I'll describe the how-to's from the perspective of a developer who skipped the 1.x knowledge base and basically wants to write operators that conforms the Reactive-Streams specification as well as RxJava 2.x's own extensions and additional expectations/requirements.

## Warning on internal components

RxJava has several hundred public classes implementing various operators and helper facilities. Since there is no way to hide these in Java 6-8, the general contract is that anything below `io.reactivex.internal` is considered private and subject to change without warnings. It is not recommended to reference these in your code (unless you contribute to RxJava itself) and must be prepared that even a patch change may shuffle/rename things around in them. That being said, they usually contain valuable tools for operator builders and as such are quite attractive to use them in your custom code.

# Atomics, serialization, deferred actions

As RxJava itself has building blocks for creating reactive dataflows, its components have building blocks as well in the form of concurrency primitives and algorithms. Many refer to the book [Concurrency in Practice](https://www.amazon.com/Java-Concurrency-Practice-Brian-Goetz/dp/0321349601) for learning the fundamentals needed. Unfortunately, other than some explanation of the Java Memory Model, the book lacks the techniques required for developing operators for RxJava 1.x and 2.x.

## Field updaters and Android

If you looked at the source code of RxJava and then at Reactor 3, you might have noticed that RxJava doesn't use the
`AtomicXFieldUpdater` classes. The reason for this is that on certain Android devices, the runtime "messes up" the field
names and the reflection logic behind the field updaters fails to locate those fields in the operators. To avoid this we decided to only use the full `AtomicX` classes (as fields or extending them).

If you target the RxJava library with your custom operator (or Android), you are ancouraged to do the same. If you plan have operators running on desktop Java, feel free to use the field updaters instead.

## Request accounting

When dealing with backpressure in `Flowable` operators, one needs a way to account the downstream requests and emissions in response to those requests. For this we use a single `AtomicLong`. Accounting must be atomic because requesting more and emitting items to fulfill an earlier request may happen at the same time.

The naive approach for accounting would be to simply call `AtomicLong.getAndAdd()` with new requests and `AtomicLong.addAndGet()` for decrementing based on how many elements were emitted.

The problem with this is that the Reactive-Streams specification declares `Long.MAX_VALUE` as the upper bound for outstanding requests (interprets it as the unbounded mode) but adding two large longs may overflow the `long` into a negative value. In addition, if for some reason, there are more values emitted than were requested, the subtraction may yield a negative current request value, causing crashes or hangs.

Therefore, both addition and subtraction have to be capped at `Long.MAX_VALUE` and `0` respectively. Since there is no dedicated `AtomicLong` method for it, we have to use a Compare-And-Set loop. (Usually, requesting happens relatively rarely compared to emission amounts thus the lack of dedicated machine code instruction is not a performance bottleneck.)

```java
public static long add(AtomicLong requested, long n) {
    for (;;) {
        long current = requested.get();
        if (current == Long.MAX_VALUE) {
            return Long.MAX_VALUE;
        }
        long update = current + n;
        if (update < 0L) {
            update = Long.MAX_VALUE;
        }
        if (requested.compareAndSet(current, update)) {
            return current;
        }
    }
}

public static long produced(AtomicLong requested, long n) {
for (;;) {
        long current = requested.get();
        if (current == Long.MAX_VALUE) {
            return Long.MAX_VALUE;
        }
        long update = current - n;
        if (update < 0L) {
            update = 0;
        }
        if (requested.compareAndSet(current, update)) {
            return update;
        }
    }
}
```

In fact, these are so common in RxJava's operators that these algorithms are available as utility methods on the **internal** `BackpressureHelper` class under the same name.

Sometimes, instead of having a separate `AtomicLong` field, your operator can extend `AtomicLong` saving on the indirection and class size. The practice in RxJava 2.x operators is that unless there is another atomic counter needed by the operator, (such as work-in-progress counter, see the later subsection) and otherwise doesn't need a base class, they extend `AtomicLong` directly.

The `BackpressureHelper` class features special versions of `add` and `produced` which treat `Long.MIN_VALUE` as a cancellation indication and won't change the `AtomicLong`s value if they see it.

## Once

RxJava 2 expanded the single-event reactive types to include `Maybe` (called as the reactive `Optional` by some). The common property of `Single`, `Completable` and `Maybe` is that they can only call one of the 3 kinds of methods on their consumers: `(onSuccess | onError | onComplete)`. Since they also participate in concurrent scenarios, an operator needs a way to ensure that only one of them is called even though the input sources may call multiple of them at once.

To ensure this, operators may use the `AtomicBoolean.compareAndSet` to atomically chose the event to relay (and thus the other events to drop).

```java
final AtomicBoolean once = new AtomicBoolean();

final MaybeObserver<? super T> child = ...; 

void emitSuccess(T value) {
   if (once.compareAndSet(false, true)) {
       child.onSuccess(value);
   }
}

void emitFailure(Throwable e) {
    if (once.compareAndSet(false, true)) {
        child.onError(e);
    } else {
        RxJavaPlugins.onError(e);
    }
}
```

Note that the same sequential requirement applies to these 0-1 reactive sources as to `Flowable`/`Observable`, therefore, if your operator doesn't have to deal with events from multiple sources (and pick one of them), you don't need this construct.

## Serialization

With more complicated sources, it may happen that multiple things happen that may trigger emission towards the downstream, such as upstream becoming available while the downstream requests for more data while the sequence gets cancelled by a timeout.

Instead of working out the often very complicated state transitions via atomics, perhaps the easiest way is to serialize the events, actions or tasks and have one thread perform the necessary steps after that. This is what I call **queue-drain** approach (or trampolining by some). 

(The other approach, **emitter-loop** is no longer recommended with 2.x due to its potential blocking `synchronized` constructs that looks performant in single-threaded case but destroys it in true concurrent case.)


The concept is relatively simple: have a concurrent queue and a work-in-progress atomic counter, enqueue the item, increment the counter and if the counter transitioned from 0 to 1, keep draining the queue, work with the element and decrement the counter until it reaches zero again:

```java
final ConcurrentLinkedQueue<Runnable> queue = ...;
final AtomicInteger wip = ...;

void execute(Runnable r) {
    queue.offer(r);
    if (wip.getAndIncrement() == 0) {
        do {
            queue.poll().run();
        } while (wip.decrementAndGet() != 0);
    }
}
```

The same pattern applies when one has to emit onNext values to a downstream consumer:

```java
final ConcurrentLinkedQueue<T> queue = ...;
final AtomicInteger wip = ...;
final Subscriber<? super T> child = ...;

void emit(T r) {
    queue.offer(r);
    if (wip.getAndIncrement() == 0) {
        do {
            child.onNext(queue.poll());
        } while (wip.decrementAndGet() != 0);
    }
}
```

### Queues

Using `ConcurrentLinkedQueue` is a reliable although mostly an overkill for such situations because it allocates on each call to `offer()` and is unbounded. It can be replaced with more optimized queues (see [JCTools](https://github.com/JCTools/JCTools/)) and RxJava itself also has some customized queues available (internal!):

  - `SpscArrayQueue` used when the queue is known to be fed by a single thread but the serialization has to look at other things (request, cancellation, termination) that can be read from other fields. Example: `observeOn` has a fixed request pattern which fits into this type of queue and extra fields for passing an error, completion or downstream requests into the drain logic.
  - `SpscLinkedArrayQueue` used when the queue is known to be fed by a single thread but there is no bound on the element count. Example: `UnicastProcessor`, almost all buffering `Observable` operator. Some operators use it with multiple event sources by synchronizing on the `offer` side - a tradeoff between allocation and potential blocking:

```java
SpscLinkedArrayQueue<T> q = ...
synchronized(q) {
    q.offer(value);
}
```

  - `MpscLinkedQueue` where there could be many feeders and unknown number of elements. Example: `buffer` with reactive boundary.

The RxJava 2.x implementations of these types of queues have different class hierarchy than the JDK/JCTools versions. Our classes don't implement the `java.util.Queue` interface but rather a custom, simplified interface:

```java
interface SimpleQueue<T> {
    boolean offer(T t);

    boolean offer(T t1, T t2);

    T poll() throws Exception;

    boolean isEmpty();

    void clear();
}

interface SimplePlainQueue<T> extends SimpleQueue<T> {
    @Override
    T poll();
}

public final class SpscArrayQueue<T> implements SimplePlainQueue<T> {
    // ...
}
```

This simplified queue API gets rid of the unused parts (iterator, collections API remnants) and adds a bi-offer method (only implemented atomically in `SpscLinkedArrayQueue` currently). The second interface, `SimplePlainQueue` is defined to suppress the `throws Exception` on poll on queue types that won't throw that exception and there is no need for try-catch around them.

## Deferred actions

The Reactive-Streams has a strict requirement that calling `onSubscribe()` must happen before any calls to the rest of the `onXXX` methods and by nature, any calls to `Subscription.request()` and `Subscription.cancel()`. The same logic applies to the design of `Observable`, `Single`, `Completable` and `Maybe` with their connection type of `Disposable`.

Often though, such call to `onSubscribe` may happen later than the respective `cancel()` needs to happen. For example, the user may want to call `cancel()` before the respective `Subscription` actually becomes available in `subscribeOn`. Other operators may need to call `onSubscribe` before they connect to other sources but at that time, there is no direct way for relaying a `cancel` call to an unavailable upstream `Subscription`.

The solution is **deferred cancellation** and **deferred requesting** in general.

(Note that some other reactive libraries, such as RxJS, mainly don't want to adopt the Reactive-Streams style of architecture because they don't seem to comprehend how to deal with this kind of "Subscription comes later but I need to cancel now" situation.)

### Deferred cancellation

This approach affects all 5 reactive types and works the same way for everyone. First, have an `AtomicReference` that will hold the respective connection type (or any other type whose method call has to happen later). Two methods are needed handling the `AtomicReference` class, one that sets the actual instance and one that calls the `cancel`/`dispose` method on it.

```java
static final Disposable DISPOSED;
static {
    DISPOSED = Disposables.empty();
    DISPOSED.dispose();
}

static boolean set(AtomicReference<Disposable> target, Disposable value) {
    for (;;) {
        Disposable current = target.get();
        if (current == DISPOSED) {
            if (value != null) {
                value.dispose();
            }
            return false;
        }
        if (target.compareAndSet(current, value)) {
            if (current != null) {
                current.dispose();
            }
            return true;
        }
    }
}

static boolean dispose(AtomicReference<Disposable> target) {
    Disposable current = target.getAndSet(DISPOSED);
    if (current != DISPOSED) {
        if (current != null) {
            current.dispose();
        }
        return true;
    }
    return false;
}
```

The approach uses an unique sentinel value `DISPOSED` - that should not appear elsewhere in your code - to indicate once a late actual `Disposable` arrives, it should be disposed immediately. Both methods return true if the operation succeeded or false if the target was already disposed.

Sometimes, only one call to `set` is permitted (i.e., `setOnce`) and other times, the previous non-null value needs no call to `dispose` because it is known to be disposed already (i.e., `replace`).

As with the request management, there are utility classes and methods for these operations:

   - (internal) `SequentialDisposable` that uses `update`, `replace` and `dispose` but leaks the API of `AtomicReference`
   - `SerialDisposable` that has safe API with `set`, `replace` and `dispose` among other things
   - (internal) `DisposableHelper` that features the methods shown above and the global disposed sentinel used by RxJava. It may come handy when one uses `AtomicReference<Disposable>` as a base class.

The same pattern applies to `Subscription` with its `cancel()` method and with helper (internal) class `SubscriptionHelper` (but no `SequentialSubscription` or `SerialSubscription`, see next subsection).

### Deferred requesting

With `Flowable`s (and Reactive-Streams `Publisher`s) the `request()` calls need to be deferred as well. In one form (the simpler one), the respective late `Subscription` will eventually arrive and we need to relay all previous and all subsequent request amount to its `request()` method.

In 1.x, this behavior was implicitly provided by `rx.Subscriber` but at a high cost that had to be payed by all instances whether or not they needed this feature.

The solution works by having the `AtomicReference` for the `Subscription` and an `AtomicLong` to store and accumulate the requests until the actual `Subscription` arrives, then atomically request all deferred value once.

```java
static boolean deferredSetOnce(AtomicReference<Subscription> subscription, 
        AtomicLong requested, Subscription newSubscription) {
    if (subscription.compareAndSet(null, newSubscription) {
        long r = requested.getAndSet(0L);
        if (r != 0) {
            newSubscription.request(r);
        }
        return true;
    }
    newSubscription.cancel();
    if (subscription.get() != SubscriptionHelper.CANCELLED) {
        RxJavaPlugins.onError(new IllegalStateException("Subscription already set!"));
    }
    return false;
}

static void deferredRequest(AtomicReference<Subscription> subscription, 
        AtomicLong requested, long n) {
    Subscription current = subscription.get();
    if (current != null) {
        current.request(n);
    } else {
        BackpressureHelper.add(requested, n);
        current = subscription.get();
        if (current != null) {
            long r = requested.getAndSet(0L);
            if (r != 0L) {
                current.request(r);
            }
        }
    }
}
```

In `deferredSetOnce`, if the CAS from null to the `newSubscription` succeeds, we atomically exchange the request amount to 0L and if the original value was nonzero, we request from `newSubscription`. In `deferredRequest`, if there is a `Subscription` we simply request from it directly. Otherwise, we accumulate the requests via the helper method then check again if the `Subscription` arrived or not. If it arrived in the meantime, we atomically exchange the accumulated request value and if nonzero, request it from the newly retrieved `Subscription`. This non-blocking logic makes sure that in case of concurrent invocations of the two methods, no accumulated request is left behind.

This complex logic and methods, along with other safeguards are available in the (internal) `SubscriptionHelper` utility class and can be used like this:

```java
final class Operator<T> implements Subscriber<T>, Subscription {
    final Subscriber<? super T> child;

    final AtomicReference<Subscription> ref = new AtomicReference<>();
    final AtomicLong requested = new AtomicLong();

    public Operator(Subscriber<? super T> child) {
        this.child = child;
    }

    @Override
    public void onSubscribe(Subscription s) {
        SubscriptionHelper.deferredSetOnce(ref, requested, s);
    }

    @Override
    public void onNext(T t) { ... }

    @Override
    public void onError(Throwable t) { ... }

    @Override
    public void onComplete() { ... }

    @Override
    public void cancel() {
        SubscriptionHelper.cancel(ref);
    }

    @Override
    public void request(long n) {
        SubscriptionHelper.deferredRequested(ref, requested, n);
    }
}

Operator<T> parent = new Operator<T>(child);

child.onSubscribe(parent);

source.subscribe(parent);
```

The second form is when multiple `Subscription`s replace each other and we not only need to hold onto request amounts when there is none of them but make sure a newer `Subscription` is requested only that much the previous `Subscription`'s upstream didn't deliver. This is called **Subscription arbitration** and the relevant algorithms are quite verbose and will be omitted here. There is, however, an utility class that manages this: (internal) `SubscriptionArbiter`.

You can extend it (to save on object headers) or have it as a field. Its main use is to send it to the downstream via `onSubscribe` and update its current `Subscription` in the current operator. Note that even though its methods are thread-safe, it is intended for swapping `Subscription`s when the current one finished emitting events. This makes sure that any newer `Subscription` is requested the right amount and not more due to production/switch race.

```java
final SubscriptionArbiter arbiter = ...

// ...

child.onSubscribe(arbiter);

// ...

long produced;

@Override
public void onSubscribe(Subscription s) {
    arbiter.setSubscription(s);
}

@Override
public void onNext(T value) {
   produced++;
   child.onNext(value);
}

@Override
public void onComplete() {
    long p = produced;
    if (p != 0L) {
        arbiter.produced(p);
    }
    subscribeNext();
}
```

For better performance, most operators can count the produced element amount and issue a single `SubscriptionArbiter.produced()` call just before switching to the next `Subscription`.


# Backpressure and cancellation

TBD

# Operator fusion

TBD

# Example implementations

TBD

## `map` + `filter` hybrid

TBD

## Ordered `merge`

TBD
