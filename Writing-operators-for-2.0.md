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


## Atomic error management

In some cases, multiple sources may signal a `Throwable` at the same time but the contract forbids calling `onError` multiple times. Once can, of course use the **once** approach with `AtomicReference<Throwable>` and throw out but the first one to set the `Throwable` on it.

The alternative is to collect these `Throwable`s into a `CompositeException` as long as possible and at one point lock out the others. This works by doing a copy-on-write scheme or by linking `CompositeException`s atomically and having a terminal sentinel to indicate all further errors should be dropped.

```java
static final Throwable TERMINATED = new Throwable();

static boolean addThrowable(AtomicReference<Throwable> ref, Throwable e) {
    for (;;) {
        Throwable current = ref.get();
        if (current == TERMINATED) {
            return false;
        }
        Throwable next;
        if (current == null) {
            next = e;
        } else {
            next = new CompositeException(current, e);
        }
        if (ref.compareAndSet(current, next)) {
            return true;
        }
    }
}

static Throwable terminate(AtomicReference<Throwable> ref) {
    return ref.getAndSet(TERMINATED);
}
```

as with most common logic, this is supported by the (internal) `ExceptionHelper` utility class and the (internal) `AtomicThrowable` class.

The usage pattern looks as follows:

```java

final AtomicThrowable errors = ...;

@Override
public void onError(Throwable e) {
    if (errors.addThrowable(e)) {
        drain();
    } else {
        RxJavaPlugins.onError(e);
    }
}

void drain() {
   // ...
   if (errors.get() != null) {
       child.onError(errors.terminate());
       return;
   }
   // ...
}
```

## Half-serialization

Sometimes having the queue-drain, `SerializedSubscriber` or `SerializedObserver` is a bit of an overkill. Such cases include when there is only one thread calling `onNext` but other threads may call `onError` or `onComplete` concurrently. Example operators include `takeUntil` where the other source may "interrupt" the main sequence and inject an `onComplete` into it before the main source itself would complete some time later. This is what I call **half-serialization**.

The approach uses the concepts of the deferred actions and atomic error management discussed above and has 3 methods for the `onNext`, `onError` and `onComplete` management:

```java
public static <T> void onNext(Subscriber<? super T> subscriber, T value,
        AtomicInteger wip, AtomicThrowable error) {
    if (wip.get() == 0 && wip.compareAndSet(0, 1)) {
        subscriber.onNext(value);
        if (wip.decrementAndGet() != 0) {
            Throwable ex = error.terminate();
            if (ex != null) {
                subscriber.onError(ex);
            } else {
                subscriber.onComplete();
            }
        }
    }
}

public static void onError(Subscriber<?> subscriber, Throwable ex,
        AtomicInteger wip, AtomicThrowable error) {
    if (error.addThrowable(ex)) {
        if (wip.getAndIncrement() == 0) {
            subscriber.onError(error.terminate());
        }
    } else {
        RxJavaPlugins.onError(ex);
    }
}

public static void onComplete(Subscriber<?> subscriber,
        AtomicInteger wip, AtomicThrowable error) {
    if (wip.getAndIncrement() == 0) {
        Throwable ex = error.terminate();
        if (ex != null) {
            subscriber.onError(ex);
        } else {
            subscriber.onComplete();
        }
    }
}
```

Here, the `wip` counter indicates there is an active emission happening and if found non-zero when trying to leave the `onNext`, it is taken as indication there was a concurrent `onError` or `onComplete()` call and the child must be notified. All subsequent calls to any of these methods are ignored. In this case, the `wip` is never decremented back to zero.

RxJava 2.x, again, supports these with the (internal) utility class `HalfSerializer` and allows targeting `Subscriber`s and `Observer`s with it.

## Fast-path queue-drain

In some operators, it is unlikely concurrent threads try to enter into the drain loop at the same time and having to play the full enqueue-increment-dequeue adds unnecessary overhead.

Luckily, such situations can be detected by a simple compare-and-set attempt on the work-in-progress counter, trying to change the amount from 0 to 1. If it fails, there is a concurrent drain in progress and we revert back to the classical queue-drain logic. If succeeds, we don't enqueue anything but emit the value / perform the action right there and we try to leave the serialized section.

```java
public void onNext(T v) {
   if (wip.get() == 0 && wip.compareAndSet(0, 1)) {
       child.onNext(v);
       if (wip.decrementAndGet() == 0) {
           break;
       }
   } else {
       queue.offer(v);
       if (wip.getAndIncrement() != 0) {
           break;
       }
   }
   drainLoop();
}

void drain() {
   if (getAndIncrement() == 0) {
       drainLoop();
   }
}

void drainLoop() {
   // the usual drain loop part after the classical getAndIncrement()
}
```

In this pattern, the classical `drain` is spit into `drain` and `drainLoop`. The new `drain` does the increment-check and calls `drainLoop` and `drainLoop` contains the remaining logic with the loop, emission and wip management as usual.

On the fast path, when we try to leave it, it is possible a concurrent call to `onNext` or `drain` incremented the `wip` counter further and the decrement didn't return it to zero. This is an indication for further work and we call `drainLoop` to process it.

# Backpressure and cancellation

Backpressure (or flow control) in Reactive-Streams is the means to tell the upstream how many elements to produce or to tell it to stop producing elements altogether. Unlike the name suggest, there is no physical pressure preventing the upstream from calling `onNext` but the protocol to honor the request amount.

In the previous section, we saw primitives to deal with request accounting and delayed `Subscriptions`, but often, operators have to react to request amount changes as well. This comes up when the operator has to decouple the downstream request amount from the amount it requests from upstream, such as `observeOn`.

Such logic can get quite complicated in operators but one of the simplest manifestation can be the `rebatchRequest` operator that combines request management with serialization to ensure that upstream is requested with a predictable pattern no matter how the downstream requested (less, more or even unbounded):

```java
final class RebatchRequests<T> extends AtomicInteger
implements Subscriber<T>, Subscription {

    final Subscriber<? super T> child;

    final AtomicLong requested;

    final SpscArrayQueue<T> queue;

    final int batchSize;

    final int limit;

    Subscription s;

    volatile boolean done;
    Throwable error;

    volatile boolean cancelled;

    long emitted;

    public RebatchRequests(Subscriber<? super T> child, int batchSize) {
        this.child = child;
        this.batchSize = batchSize;
        this.limit = batchSize - (batchSize >> 2); // 75% of batchSize
        this.requested = new AtomicLong();
        this.queue = new SpscArrayQueue<T>(batchSize);
    }

    @Override
    public void onSubscribe(Subscription s) {
        this.s = s;
        child.onSubscribe(this);
        s.request(batchSize);
    }

    @Override
    public void onNext(T t) {
        queue.offer(t);
        drain();
    }

    @Override
    public void onError(Throwable t) {
        error = t;
        done = true;
        drain();
    }

    @Override
    public void onComplete() {
        done = true;
        drain();
    }

    @Override
    public void request(long n) {
        BackpressureHelper.add(requested, n);
        drain();
    }

    @Override
    public void cancel() {
        cancelled = true;
        s.cancel();
    }

    void drain() {
        // see next code example
    }
}
```

Here we extend `AtomicInteger` since the work-in-progress counting happens more often and is worth avoiding the extra indirection. The class extends `Subscription` and it hands itself to the `child` `Subscriber` to capture its `request()`  (and `cancel()`) calls and route it to the main `drain` logic. Some operators need only this, some other operators (such as `observeOn` not only routes the downstream request but also does extra cancellations (cancels the asynchrony providing `Worker` as well) in its `cancel()` method.

**Important**: when implementing operators for `Flowable` and `Observable` in RxJava 2.x, you are not allowed to pass along an upstream `Subscription` or `Disposable` to the child `Subscriber`/`Observer` when the operator logic itself doesn't require hooking the `request`/`cancel`/`dispose` calls. The reason for this is how operator-fusion is implemented on top of `Subscription` and `Disposable` passing through `onSubscribe` in RxJava 2.x (and in Reactor 3). See the next section about operator-fusion. There is no fusion in `Single`, `Completable` or `Maybe` (because there is no requesting or unbounded buffering with them) and their operators can pass the upstream `Disposable` along as is.

Next comes the `drain` method whose pattern appears in many operators (with slight variations on how and what the emission does).

```java
void drain() {
    if (getAndIncrement() != 0) {
        return;
    }

    int missed = 1;

    for (;;) {
        long r = requested.get();
        long e = 0L;
        long f = emitted;
        
        while (e != r) {
            if (cancelled) {
                return;
            }
            boolean d = done;

            if (d) {
                Throwable ex = error;
                if (ex != null) {
                    child.onError(ex);
                    return;
                }
            }

            T v = queue.poll();
            boolean empty = v == null;

            if (d && empty) {
                child.onComplete();
                return;
            }

            if (empty) {
                break;
            }

            child.onNext(v);
            
            e++;
            if (++f == limit) {
               s.request(f);
               f = 0L;
            }
        }

        if (e != r) {
            if (cancelled) {
                return;
            }

            if (done) {
                Throwable ex = error;
                if (ex != null) {
                    child.onError(ex);
                    return;
                }
                if (queue.isEmpty()) {
                    child.onComplete();
                    return;
                }
            }
        }

        if (e != 0L) {
            BackpressureHelper.produced(requested, e);
        }

        emitted = f;
        missed = addAndGet(-missed);
        if (missed == 0) {
            break;
        }
    }
}
```

This particular pattern is called the **stable-request queue-drain**. Another variation doesn't care about request amount stability towards upstream and simply requests the amount it delivered to the child:

```java
void drain() {
    if (getAndIncrement() != 0) {
        return;
    }

    int missed = 1;

    for (;;) {
        long r = requested.get();
        long e = 0L;
        
        while (e != r) {
            if (cancelled) {
                return;
            }
            boolean d = done;

            if (d) {
                Throwable ex = error;
                if (ex != null) {
                    child.onError(ex);
                    return;
                }
            }

            T v = queue.poll();
            boolean empty = v == null;

            if (d && empty) {
                child.onComplete();
                return;
            }

            if (empty) {
                break;
            }

            child.onNext(v);
            
            e++;
        }

        if (e != r) {
            if (cancelled) {
                return;
            }

            if (done) {
                Throwable ex = error;
                if (ex != null) {
                    child.onError(ex);
                    return;
                }
                if (queue.isEmpty()) {
                    child.onComplete();
                    return;
                }
            }
        }

        if (e != 0L) {
            BackpressureHelper.produced(requested, e);
            s.request(e);
        }

        missed = addAndGet(-missed);
        if (missed == 0) {
            break;
        }
    }
}
```

The third variation allows delaying a potential error until the upstream has terminated and all normal elements have been delivered to the child:

```java
final boolean delayError;

void drain() {
    if (getAndIncrement() != 0) {
        return;
    }

    int missed = 1;

    for (;;) {
        long r = requested.get();
        long e = 0L;
        
        while (e != r) {
            if (cancelled) {
                return;
            }
            boolean d = done;

            if (d && !delayError) {
                Throwable ex = error;
                if (ex != null) {
                    child.onError(ex);
                    return;
                }
            }

            T v = queue.poll();
            boolean empty = v == null;

            if (d && empty) {
                Throwable ex = error;
                if (ex != null) {
                    child.onError(ex);
                } else {
                    child.onComplete();
                }
                return;
            }

            if (empty) {
                break;
            }

            child.onNext(v);
            
            e++;
        }

        if (e != r) {
            if (cancelled) {
                return;
            }

            if (done) {
                if (delayError) {
                    if (queue.isEmpty()) {
                        Throwable ex = error;
                        if (ex != null) {
                            child.onError(ex);
                        } else {
                            child.onComplete();
                        }
                        return;
                    }
                } else {
                    Throwable ex = error;
                    if (ex != null) {
                        child.onError(ex);
                        return;
                    }
                    if (queue.isEmpty()) {
                        child.onComplete();
                        return;
                    }
                }
            }
        }

        if (e != 0L) {
            BackpressureHelper.produced(requested, e);
            s.request(e);
        }

        missed = addAndGet(-missed);
        if (missed == 0) {
            break;
        }
    }
}
```

If the downstream cancels the operator, the `queue` may still hold elements which may get referenced longer than expected if the operator chain itself is referenced in some way. On the user level, applying `onTerminateDetach` will forget all references going upstream and downstream and can help with this situation. On the operator level, RxJava 2.x usually calls `clear()` on the `queue` when the sequence is cancelled or ends before the queue is drained naturally. This requires some slight change to the drain loop:

```java
final boolean delayError;

@Override
public void cancel() {
    cancelled = true;
    s.cancel();
    if (getAndIncrement() == 0) {
        queue.clear();    // <----------------------------
    }
}

void drain() {
    if (getAndIncrement() != 0) {
        return;
    }

    int missed = 1;

    for (;;) {
        long r = requested.get();
        long e = 0L;
        
        while (e != r) {
            if (cancelled) {
                queue.clear();    // <----------------------------
                return;
            }
            boolean d = done;

            if (d && !delayError) {
                Throwable ex = error;
                if (ex != null) {
                    queue.clear();    // <----------------------------
                    child.onError(ex);
                    return;
                }
            }

            T v = queue.poll();
            boolean empty = v == null;

            if (d && empty) {
                Throwable ex = error;
                if (ex != null) {
                    child.onError(ex);
                } else {
                    child.onComplete();
                }
                return;
            }

            if (empty) {
                break;
            }

            child.onNext(v);
            
            e++;
        }

        if (e != r) {
            if (cancelled) {
                queue.clear();    // <----------------------------
                return;
            }

            if (done) {
                if (delayError) {
                    if (queue.isEmpty()) {
                        Throwable ex = error;
                        if (ex != null) {
                            child.onError(ex);
                        } else {
                            child.onComplete();
                        }
                        return;
                    }
                } else {
                    Throwable ex = error;
                    if (ex != null) {
                        queue.clear();    // <----------------------------
                        child.onError(ex);
                        return;
                    }
                    if (queue.isEmpty()) {
                        child.onComplete();
                        return;
                    }
                }
            }
        }

        if (e != 0L) {
            BackpressureHelper.produced(requested, e);
            s.request(e);
        }

        missed = addAndGet(-missed);
        if (missed == 0) {
            break;
        }
    }
}
```

Since the queue is single-producer-single-consumer, its `clear()` must be called from a single thread - which is provided by the serialization loop and is enabled by the `getAndIncrement() == 0` "half-loop" inside `cancel()`.

An important note on the order of calls to `done` and the `queue`'s state:

```java
boolean d = done;
T v = queue.poll();
```

and

```java
boolean d = done;
boolean empty = queue.isEmpty();
```

These must happen in the order specified. If they were swapped, it is possible when the drain runs asynchronously to an `onNext`/`onComplete()`, the queue may appear empty at first, then it gets elements followed by `done = true` and a late `done` check in the drain loop may complete the sequence thinking it delivered all values there was.

## Single valued results

Sometimes an operator only emits one single value at some point instead of emitting more or all of its sources. Such operators include `fromCallable`, `reduce`, `any`, `all`, `first`, etc.

The classical queue-drain works here but is a bit of an overkill to allocate objects to store the work-in-progress counter, request accounting and the queue itself. These elements can be reduced to a single state-machine with one state counter object - often inlinded by extending AtomicInteger - and a plain field for storing the single value to be emitted.

The state machine handing the possible concurrent downstream requests and normal completion path is a bit complicated to show here and is quite easy to get wrong.

RxJava 2.x supports this kind of behavior through the (internal) `DeferredScalarSubscription` for operators without an upstream source (`fromCallable`) and the (internal) `DeferredScalarSubscriber` for reduce-like operators with an upstream source.

Using the `DeferredScalarSubscription` is straightforward, one creates it, sends it to the downstream via `onSubscribe` and later on calls `complete(T)` to signal the end with a single value:

```java
DeferredScalarSubscription<Integer> dss = new DeferredScalarSubscription<>(child);
child.onSubscribe(dss);

dss.complete(1);
```

Using the `DeferredScalarSubscriber` requires more coding and extending the class itself:

```java
final class Counter extends DeferredScalarSubscriber<Object, Integer> {
   public Counter(Subscriber<? super Integer> child) {
       super(child);
       value = 0;
       hasValue = true;
   }

   @Override
   public void onNext(Object t) {
       value++;
   }
}
```

By default, the `DeferredScalarSubscriber.onSubscribe()` requests `Long.MAX_VALUE` from the upstream (but the method can be overridden in subclasses).

## Single-element post-complete

TBD

## Multi-element post-complete

TBD

# Creating operator classes

Creating operator implementations in 2.x is simpler than in 1.x and incurs less allocation as well. You have the choice to implement your operator as a `Subscriber`-transformer to be used via `lift` or as a fully-fledged base reactive class.

## Operator by extending a base reactive class

In 1.x, extending `Observable` was possible but convoluted because you had to implement the `OnSubscribe` interface separately and pass it to `Observable.create()` or to the `Observable(OnSubscribe)` protected constructor.

In 2.x, all base reactive classes are abstract and you can extend them directly without any additional indirection:

```java
public final class FlowableMyOperator extends Flowable<Integer> {
    final Publisher<Integer> source;
 
    public FlowableMyOperator(Publisher<Integer> source) {
        this.source = source;
    }
    
    @Override
    protected void subscribeActual(Subscriber<? super Integer> s) {
        source.map(v -> v + 1).subscribe(s);
    }
}
```

When taking other reactive types as inputs in these operators, it is recommended one defines the base reactive interfaces instead of the abstract classes, allowing better interoperability between libraries (especially with `Flowable` operators and other Reactive-Streams `Publisher`s). To recap, these are the class-interface pairs:

  - `Flowable` - `Publisher` - `Subscriber`
  - `Observable` - `ObservableSource` - `Observer`
  - `Single` - `SingleSource` - `SingleObserver`
  - `Completable` - `CompletableSource` - `CompletableObserver`
  - `Maybe` - `MaybeSource` - `MaybeObserver`

RxJava 2.x locks down `Flowable.subscribe` (and the same methods in the other types) in order to provide runtime hooks into the various flows, therefore, implementors are given the `subscribeActual()` to be overridden. When it is invoked, all relevant hooks and wrappers have been applied. Implementors should avoid throwing unchecked exceptions as the library generally can't deliver it to the respective `Subscriber` due to lifecycle restrictions of the Reactive-Streams specification and sends it to the global error consumer via `RxJavaPlugins.onError`.

Unlike in 1.x, In the example above, the incoming `Subscriber` is simply used directly for subscribing again (but still at most once) without any kind of wrapping. In 1.x, one needs to call `Subscribers.wrap` to avoid double calls to `onStart` and cause unexpected double initialization or double-requesting.

Unless one contributes a new operator to RxJava, working with such classes may become tedious, especially if they are intermediate operators:

```java
new FlowableThenSome(
    new FlowableOther(
        new FlowableMyOperator(Flowable.range(1, 10).map(v -> v * v))
    )
)
```

This is an unfortunate effect of Java lacking extension method support. A possible ease on this burden is by using `compose` to have fluent inline application of the custom operator:

```java
Flowable.range(1, 10).map(v -> v * v)
.compose(f -> new FlowableOperatorWithParameter(f, 10));

Flowable.range(1, 10).map(v -> v * v)
.compose(FlowableMyOperator::new);
```

## Operator targeting lift()

The alternative to the fluent application problem is to have a `Subscription`-transformer implemented instead of extending the whole reactive base class and use the respective type's `lift()` operator to get it into the sequence.

First one has to implement the respective `XOperator` interface:

```java
public final class MyOperator implements FlowableOperator<Integer, Integer> {

    @Override
    public Subscriber<? super Integer> apply(Subscriber<? super Integer> child) {
    }

    static final class Op implements Subscriber<Integer>, Subscription {
        final Subscriber<? super Integer> child;

        @Override
        public Op(Subscriber<? super Integer> child) {
            this.child = child;
        }


    }
}
```

You may recognize that implementing operators via extension or lifting looks quite similar. In both cases, one usually implements a `Subscriber` (`Observer`, etc) that takes a downstream `Subscriber`, implements the business logic in the `onXXX` methods and somehow (manually or as part of `lift()`'s lifecycle) gets subscribed to an upstream source.

# Operator fusion

Operator fusion has the premise that certain operators can be combined into one single operator (macro-fusion) or their internal data structures shared between each other (micro-fusion) that allows fewer allocations, lower overhead and better performance.

This advanced concept was invented, worked out and studied in the [Reactive-Streams-Commons](https://github.com/reactor/reactive-streams-commons) research project manned by the leads of RxJava and Project Reactor. Both libraries use the results in their implementation, which look the same but are incompatible due to different classes and packages involved. In addition, RxJava 2.x's approach is a more polished version of the invention due to delays between the two project's development.

Given this novel approach, a generation number can be assigned to various implementation styles of reactive libraries:

0. These are the classical libraries that either use `java.util.Observable` or are listener based (Java Swing's `ActionListener`). Their common property is that they don't support composition (of events and cancellation).
1. This is the classical Rx.NET library that supports composition, but has no notion for backpressure and doesn't properly support synchronous cancellation.
2. This is what RxJava 1.x is categorized, it supports composition, backpressure and synchronous cancellation along with the ability to lift an operator into a sequence.
3. This is the level of the Reactive-Streams based libraries such as Reactor 2 and Akka-Stream. They are based upon a specification that evolved out of RxJava but left behind its drawbacks (such as the need to return anything from `subscribe()`).
4. This level expands upon the Reactive-Streams interfaces with operator-fusion (in a compatible fashion)

## Components

TBD

### QueueSubscription and QueueDisposable

TBD

### ConditionalSubscriber

TBD

# Example implementations

TBD

## `map` + `filter` hybrid

TBD

## Ordered `merge`

TBD
