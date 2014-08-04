_**Work in progress...**_

In RxJava it is not difficult to get into a situation in which an Observable is emitting items more rapidly than an operator or subscriber can consume them. This presents the problem of what to do with such a growing backlog of unconsumed items.

For example, imagine using the [`zip`](Combining-Observables#wiki-zip) operator to zip together two infinite Observables, one of which emits items twice as frequently as the other. The `zip` operator, to perform as advertised, will have to maintain an ever-expanding buffer of items emitted by the faster Observable to eventually combine with items emitted by the slower one. This could cause RxJava to seize an unwieldy amount of system resources.

There are a variety of strategies with which you can exercise flow control and backpressure in RxJava. This page explains some of these strategies, and also shows you how you can design your own Observables and Observable operators to respect requests for flow control.

## Useful operators that avoid the need for backpressure

Your first line of defense against the problems of over-producing Observables is the ordinary set of Observable operators. The examples in this section will show how you might use these operators to handle a bursty Observable like the one illustrated in the following marble diagram:
<img src="/Netflix/RxJava/wiki/images/rx-operators/bp.bursty.png" width="640" height="35" />​

### Throttling

Operators like [`sample( )` or `throttleLast( )`](Filtering-Observables#wiki-sample-or-throttlelast), [`throttleFirst( )`](Filtering-Observables#wiki-throttlefirst), and [`throttleWithTimeout( )` or `debounce( )`](Filtering-Observables#wiki-throttlewithtimeout-or-debounce) allow you to regulate the rate at which an Observable emits items.

The following diagrams show how you could use each of these operators on the bursty Observable shown above.

#### sample (or throttleLast)
The `sample` operator periodically "dips" into the sequence and emits only the most recently emitted item during each dip:
<img src="/Netflix/RxJava/wiki/images/rx-operators/bp.sample.png" width="640" height="260" />​
````groovy
Observable<Integer> burstySampled = bursty.sample(500, TimeUnit.MILLISECONDS);
````

#### throttleFirst
The `throttleFirst` operator is similar, but emits not the most recently emitted item, but the first item that was emitted after the previous "dip":
<img src="/Netflix/RxJava/wiki/images/rx-operators/bp.throttleFirst.png" width="640" height="260" />​
````groovy
Observable<Integer> burstyThrottled = bursty.throttleFirst(500, TimeUnit.MILLISECONDS);
````

#### debounce (or throttleWithTimeout)
The `debounce` operator only emits those items from the source Observable that are not followed by another item within a specified duration:
<img src="/Netflix/RxJava/wiki/images/rx-operators/bp.debounce.png" width="640" height="240" />​
````groovy
Observable<Integer> burstyDebounced = bursty.debounce(10, TimeUnit.MILLISECONDS);
````

### Buffers and windows

You could also use an operator like [`buffer( )`](Transforming-Observables#buffer) or [`window( )`](Transforming-Observables#window) to collect items from the over-producing Observable and then emit them, less-frequently, as collections (or Observables) of items. The slow consumer can then decide whether to process only one particular item from each collection, to process some combination of those items, or to schedule work to be done on each item in the collection, as appropriate.

The following diagrams show how you could use each of these operators on the bursty Observable shown above.

#### buffer

You could, for example, close and emit a buffer of items from the bursty Observable periodically, at a regular interval of time:
<img src="/Netflix/RxJava/wiki/images/rx-operators/bp.buffer2.png" width="640" height="270" />​
````groovy
Observable<List<Integer>> burstyBuffered = bursty.buffer(500, TimeUnit.MILLISECONDS);
````

Or you could get fancy, and collect items in buffers during the bursty periods and emit them at the end of each burst, by using the `debounce` operator to emit a buffer closing indicator to the `buffer` operator:
<img src="/Netflix/RxJava/wiki/images/rx-operators/bp.buffer1.png" width="640" height="500" />​
````groovy
// we have to multicast the original bursty Observable so we can use it both as our source and as the
// ultimate source for our buffer closing selector:
Observable<Integer> burstyMulticast = bursty.publish().refCount();
// burstyDebounced will be our buffer closing selector:
Observable<Integer> burstyDebounced = burstMulticast.debounce(10, TimeUnit.MILLISECONDS);
// and this, finally, is the Observable of buffers we're interested in:
Observable<List<Integer>> burstyBuffered = burstyMulticast.buffer(burstyDebounced);
````

#### window

`window`, like `buffer`, allows you to periodically emit Observable windows of items at a regular interval of time:

<img src="/Netflix/RxJava/wiki/images/rx-operators/bp.window1.png" width="640" height="325" />​
````groovy
Observable<Observable<Integer>> burstyWindowed = bursty.window(500, TimeUnit.MILLISECONDS);
````

You could also choose to emit a new window each time you have collected a particular number of items from the source Observable:

<img src="/Netflix/RxJava/wiki/images/rx-operators/bp.window2.png" width="640" height="325" />​
````groovy
Observable<Observable<Integer>> burstyWindowed = bursty.window(5);
````

## Callstack blocking as a flow-control alternative to backpressure

Another way of handling an overproductive Observable is to block the callstack (parking the thread that governs the overproductive Observable). This has the disadvantage of going against the “reactive” and non-blocking model of Rx. However this can be a viable option if the problematic Observable is on a thread that can be blocked safely. Currently RxJava does not expose any operators to facilitate this.

If the Observable, all of the operators that operate on it, and the observer that is subscribed to it, are all operating in the same thread, this effectively establishes a form of backpressure by means of callstack blocking. But be aware that many Observable operators do operate in their own threads by default (the javadocs for those operators will indicate this).

## Backpressure isn’t magic

Backpressure doesn’t make the problem of an overproducing Observable or an underconsuming Subscriber go away. It just moves the problem up the chain of operators to a point where it can be handled better.

Let’s take a closer look at the problem of the uneven `zip`.

You have two Observables, _A_ and _B_, where _B_ is inclined to emit items more frequently as _A_. When you try to `zip` these two Observables together, the zip operator combines item _n_ from _A_ and item _n_ from _B_, but meanwhile _B_ has also emitted items _n_+1 to _n_+_m_. The zip operator has to hold on to these items so it can combine them with items _n_+1 to _n_+_m_ from _A_ as they are emitted, but meanwhile _m_ keeps growing and so the size of the buffer needed to hold on to these items keeps increasing.

You could attach a throttling operator to _B_, but this would mean ignoring some of the items _B_ emits, which might not be appropriate. What you’d really like to do is to signal to _B_ that it needs to slow down and then let _B_ decide how to do this in a way that maintains the integrity of its emissions.

The reactive pull backpressure model lets you do this.  The `Subscriber` interface has a method called `request()` that lets it ask for a specified number of items from the Observable the Subscriber is subscribed to.  A `Subscriber` can call this method inside its `onStart()` handler to initiate the emission of items, and in its `onNext()` handler to keep the flow of emissions coming.  This creates a sort of active pull from the Subscriber in contrast to the normal passive push Observable behavior.

The `zip` operator in RxJava uses this technique. It maintains a small buffer of items, and requests no more from its source Observables than would fill that buffer. Each time `zip` emits an item, it removes that item from its buffer and requests exactly one more item from each of its source Observables.

(Most RxJava operators exercise this variety of backpressure. Some operators do not need to use this variety of backpressure, as they operate in the same thread as the Observable they operate on, and so they exert a form of blocking backpressure simply by not giving the Observable the opportunity to emit another item until they have finished processing the previous one. For other operators, backpressure is inappropriate as they have been explicitly designed to deal with flow control in other ways. The RxJava javadocs for those operators that are methods of the Observable class indicate which ones do not use standard backpressure and why.)

For this to work, though, _A_ and _B_ (or the Observables that result from operators applied to them) must respond correctly to the `request()`.  If an Observable has not been written to support backpressure (such support is not a requirement for Observables), you can apply one of the following operators to it, each of which forces a simple form of backpressure behavior:

<dl>
 <dt><tt>onBackpressureBuffer</tt></dt>
  <dd>maintains a buffer of all emissions from the source Observable and emits them to downstream Subscribers according to the <tt>request</tt>s they generate</dd>
 <dt><tt>onBackpressureDrop</tt></dt>
  <dd>drops emissions from the source Observable unless there is a pending <tt>request</tt> from a downstream Subscriber, in which case it will emit enough items to fulfill the request</dd>
</dl>

If you do not apply either of these operators to an Observable that does not support backpressure, _and_ if either you as the Subscriber or some operator between you and the Observable attempts to apply backpressure, you will encounter a `MissingBackpressureException` which you will be notified of via your `onError()` callback.

## How to request backpressure from a subscriber

When you subscribe to an `Observable` with a `Subscriber`, you can request backpressure by calling `Subscriber.request(n)` in the `Subscriber`&#8217;s `onStart()` method, where _n_ is the maximum number of items you want the `Observable` to emit before the next `request()` call.

Then, after handling this item (or these items) in `onNext()`, you can call `request()` again to instruct the `Observable` to emit another item (or items).  Here is an example of a `Subscriber` that requests one item at a time from `someObservable`:

````java
someObservable.subscribe(new Subscriber<t>() {
    @Override
    public void onStart() {
      request(1);
    }

    @Override
    public void onCompleted() {
      // gracefully handle sequence-complete
    }

    @Override
    public void onError(Throwable e) {
      // gracefully handle error
    }

    @Override
    public void onNext(t n) {
      // do something with the emitted item "n"
      // request another item:
      request(1);
    }
});
````

# Hot and cold Observables, and multicasted Observables

A _cold_ Observable emits a particular sequence of items, but can begin emitting this sequence at any time its Observer finds convenient, and at whatever rate the Observer finds convenient, without disrupting the integrity of the sequence. For example if you convert a static Iterable into an Observable, that Observable will emit the same sequence of items no matter when it is later subscribed to or how frequently those items are observed.

A _hot_ Observable begins generating items to emit immediately when it is created. Subscribers typically begin observing the sequence of items emitted by a hot Observable from somwhere in the middle of the sequence, beginning with the first item emitted by the Observable subsequent to the establishment of the subscription. Such an Observable emits items at its own pace, and it is up to its Observables to keep up.

When a cold Observable is _multicast_ (when it is converted into a `ConnectableObservable` and its `connect()` method is called), it effectively becomes _hot_ and for the purposes of backpressure and flow-control should be treated as a hot Observable.

Cold observables are ideal subjects for the reactive pull model of backpressure described above. Hot observables are typically not designed to cope well with a reactive pull model, and are better candidates for some of the other strategies discussed on this page, such as the use of the `onBackpressureBuffer` or `onBackpressureDrop` operators, throttling, buffers, or windows.

_**Work in progress...**_

Things that may need explaining:
* the `Producer` interface (and its `request` method)
* other new method in `Subscriber`:
  * `setProducer(p)`
* how and when to support producers in custom observables & operators
  * point here from the "how to make a custom operator" page; maybe also from `create` operator doc

_**Work in progress...**_

## See also
* [RxJava 0.20.0-RC1 release notes](https://github.com/Netflix/RxJava/releases/tag/0.20.0-RC1)