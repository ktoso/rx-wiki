If you want to introduce multithreading into your cascade of Observable operators, you can do so by instructing those operators (or particular Observables) to operate on particular Schedulers.

You can make an Observable act on a particular Scheduler by means of the <a href="https://github.com/Netflix/RxJava/wiki/Observable-Utility-Operators#wiki-observeon">`observeOn`</a> and <a href="https://github.com/Netflix/RxJava/wiki/Observable-Utility-Operators#wiki-subscribeon">`subscribeOn`</a> operators. You can also split an operator that works on an Observable onto multiple threads with the <a href="https://github.com/Netflix/RxJava/wiki/Observable-Utility-Operators#wiki-parallel">`parallel`</a> operator.

Many of the RxJava Observable operators have varieties that take a Scheduler as a parameter. These instruct the operator to do some or all of its work on a particular Scheduler.

#### See also:
* <a href="http://www.introtorx.com/Content/v1.0.10621.0/15_SchedulingAndThreading.html">Intro to Rx: Scheduling and Threading</a>

## Varieties of Scheduler

You obtain a Scheduler from the factory methods described in the `Schedulers` class. The following table shows the varieties of Scheduler that are available to you by means of these methods:

<table>
 <thead>
  <tr><th><code>Scheduler</cote></th><th>purpose</th></tr>
 </thead>
 <tbody>
  <tr><td><code>Schedulers.computation(&#8239;)</code></td><td>meant for computational work such as event-loops and callback processing; do not use this scheduler for I/O (use <code>Schedulers.io(&#8239;)</code> instead)</td></tr>
  <tr><td><code>Schedulers.trampoline(&#8239;)</code></td><td>queues work to begin on the current thread after any already-queued work</td></tr>
  <tr><td><code>Schedulers.executor(&#8239;)</code></td><td>queues work to be done on either an <code>Executor</code> or <code>ScheduledExecutorService</code> (Note that if you use an <code>Executor</code> instead of a <code>ScheduledExecutorService</code> then the Scheduler will use a system-wide <code>Timer</code> to handle delayed events.)</td></tr>
  <tr><td><code>Schedulers.immediate(&#8239;)</code></td><td>schedules work to begin immediately in the current thread</td></tr>
  <tr><td><code>Schedulers.io(&#8239;)</code></td><td>meant for I/O-bound work such as asynchronous performance of blocking I/O, this scheduler is backed by an Executor thread-pool that will grow as needed; for ordinary computational work, switch to <code>Schedulers.computation(&#8239;)</code></td></tr>
  <tr><td><code>Schedulers.newThread(&#8239;)</code></td><td>creates a new thread for each unit of work</td></tr>
 </tbody>
</table>
## Default Schedulers for RxJava Observable operators

Some Observable operators in RxJava have alternate forms that allow you to set which Scheduler the operator will use for (at least some part of) its operation. For these operators, if you do not set the Scheduler, the operator will use the default `computation` Scheduler.

Other operators do not have a form that permits you to set their Schedulers. Some of these, like `startWith`, `empty`, `error`, `from`, `just`, `merge`, and `range` do not use a Scheduler. A few others use particular schedulers, as in the following table:
<table>
 <thead>
  <tr><th>operator</th><th>Scheduler</th></tr>
 </thead>
 <tbody>
  <tr><td><code>parallelMerge</code></td><td><code>currentThread</code></td></tr>
  <tr><td><code>repeat</code></td><td><code>currentThread</code></td></tr>
  <tr><td><code>timeInterval</code></td><td><code>immediate</code></td></tr>
  <tr><td><code>timestamp</code></td><td><code>immediate</code></td></tr>
 </tbody>
</table>

## Using Schedulers

Aside from passing these Schedulers in to RxJava Observable operators, you can also use them to schedule your own work. The following example uses the `schedule( )` method of the `Scheduler` class to schedule work on the `newThread` Scheduler:

```java
Schedulers.newThread().schedule(new Action1<Inner>() {

    @Override
    public void call(Inner inner) {
        doWork();
    }

});
```
The `inner` parameter allows you to schedule recursive calls:
```java
Schedulers.newThread().schedule(new Action1<Inner>() {

    @Override
    public void call(Inner inner) {
        doWork();
        // recurse until unsubscribed (the schedule will do nothing if unsubscribed)
        inner.schedule(this);
    }

});
```
`inner` also implements the `Subscription` interface, and its `isUnsubscribed( )` and `unsubscribe( )` methods, so you can stop work when a subscription is cancelled, or you can cancel the subscription from within the scheduled task:
```java
Schedulers.newThread().schedule(new Action1<Inner>() {

    @Override
    public void call(Inner inner) {
        while(!inner.isUnsubscribed()) {
            status = doWork();
            if(QUIT == status) { inner.unsubscribe(); }
        }
    }

});
```
The `schedule( )` method returns a `Subscription` and so you can call its `unsubscribe( )` method to signal that it can halt work.

You can also use a version of `schedule( )` that delays your task on the given Scheduler until a certain timespan has passed. The following example schedules `someTask` to be performed on `someScheduler` after 500ms have passed according to that Scheduler's clock:
```java
someScheduler.schedule(someTask, 500, TimeUnit.MILLISECONDS);
```