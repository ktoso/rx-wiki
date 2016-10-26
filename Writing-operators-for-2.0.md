# Introduction

Writing operators, source-like (`fromAsync`) or intermediate-like (`flatMap`) **has always been a hard task to do in RxJava**. There are many rules to obey, many cases to consider but at the same time, many (legal) shortcuts to take to build a well performing code. Now writing an operator specifically for 2.x is 10 times harder than for 1.x. If you want to exploit all the advanced, 4th generation features, that's even 2-3 times harder on top (so 30 times harder in total).

*(If you have been following [my blog](http://akarnokd.blogspot.hu/) about RxJava internals, writing operators is maybe only 2 times harder than 1.x; some things have moved around, some tools popped up while others have been dropped but there is a relatively straight mapping from 1.x concepts and approaches to 2.x concepts and approaches.)*

In this article, I'll describe the how-to's from the perspective of a developer who skipped the 1.x knowledge base and basically wants to write operators that conforms the Reactive-Streams specification as well as RxJava 2.x's own extensions and additional expectations/requirements.

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
public static long addCap(AtomicLong requested, long n) {
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

Sometimes, instead of having a separate `AtomicLong` field, your operator can extend `AtomicLong` saving on the indirection and class size. The practice in RxJava 2.x operators is that unless there is another atomic counter needed by the operator, (such as work-in-progress counter, see next subsection) and otherwise doesn't need a base class, they extend `AtomicLong` directly.

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
