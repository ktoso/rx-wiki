_**Work in progress...**_

In RxJava it is not difficult to get into a situation in which an Observable is emitting items more rapidly than an operator or subscriber can consume them. This presents the problem of what to do with such a growing backlog of unconsumed items.

For example, imagine using the [`zip`](Combining-Observables#wiki-zip) operator to zip together two infinite Observables, one of which emits items twice as frequently as the other. The `zip` operator, to perform as advertised, will have to maintain an ever-expanding buffer of items emitted by the faster Observable to combine with items emitted by the slower one. This could cause RxJava to seize an unwieldy amount of system resources.

You can tell RxJava how you want it to handle cases like these. RxJava is capable of exerting _backpressure_ on Observables. This page tells you how the backpressure options work, and also how you can design your own Observables and Observable operators to respect backpressure requests.

## Useful Operators that Avoid the Need for Backpressure

Your first line of defense against the problems of over-producing Observables is the ordinary set of Observable operators. In particular, operators like [`sample( )` or `throttleLast( )`](Filtering-Observables#wiki-sample-or-throttlelast), [`throttleFirst( )`](Filtering-Observables#wiki-throttlefirst), and [`throttleWithTimeout( )` or `debounce( )`](Filtering-Observables#wiki-throttlewithtimeout-or-debounce) allow you to regulate the rate at which an Observable emits items.

We might, for example, have used one of these operators on each of the two Observables we intended to `zip` together in the conundrum mentioned earlier, and this would have solved our problem.  But the behavior of the resulting `zip` would also have been different. It would no longer necessarily zip together the <i>n</i><sup>th</sup> item from each Observable sequentially.

## Backpressure Isn't Magic

Backpressure doesn't make the problem of an overproducing Observable or an underconsuming Subscriber go away. It just moves the problem up the chain of operators to a point where it can be handled better.

Let's take a closer look at the problem of the uneven `zip`.

You have two Observables, _A_ and _B_, where _B_ is inclined to emit items more frequently as _A_. When you try to `zip` these two Observables together, the zip operator combines item _n_ from _A_ and item _n_ from _B_, but meanwhile _B_ has also emitted items _n_+1 to _n_+_m_. The zip operator has to hold on to these items so it can combine them with items _n_+1 to _n_+_m_ from _A_ as they are emitted, but meanwhile _m_ keeps growing and so the size of the buffer needed to hold on to these items keeps increasing.

You could attach a throttling operator to _B_, but this would mean ignoring some of the items _B_ emits, which might not be appropriate. What you'd really like to do is to signal to _B_ that it needs to slow down and then let _B_ decide how to do this in a way that maintains the integrity of its emissions.

Backpressure lets you do this.  The `Subscriber` interface has a method called `request(_n_)` that lets it ask for a specified number of items from the Observable the Subscriber is subscribed to.  A `Subscriber` can call this method inside its `onStart()` handler to initiate the emission of items and in its `onNext()` handler to keep the flow of emissions coming.  This creates a sort of active pull from the Subscriber in contrast to the normal passive push Observable behavior.

In our `zip` example, we could tell `zip` to request one item from both _A_ and _B_ only when `zip` itself emits an item; that way `zip` would never have to buffer items from a more prolific Observable.

For this to work, though, _A_ and _B_ (or the Observables that result from operators applied to them) must respond correctly to the `request()`.  If an Observable has not been written to support backpressure, you can apply one of the following operators to it, each of which forces a simple form of backpressure behavior:

<dl>
 <dt><tt>onBackpressureBuffer</tt></dt>
  <dd>maintains a buffer of all emissions from the source Observable and emits them to downstream Subscribers according to the <tt>request</tt>s they generate</dd>
 <dt><tt>onBackpressureDrop</tt></dt>
  <dd>drops emissions from the source Observable unless there is a pending <tt>request</tt> from a downstream Subscriber, in which case it will emit enough items to fulfill the request</dd>
</dl>

## How to request backpressure in a Subscriber

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

_**Work in progress...**_

Things that may need explaining:
* the `Producer` interface (and its `request` method)
  * how to request w/o backpressure: `request(Long.MAX_VALUE)`
* the new methods in `Subscriber`
  * `request(n)`
  * `setProducer(p)`
  * `onStart()`
  * `onSetProducer(p)`
  * and the new constructors that take a `bufferRequest` parameter
* the new `Observable` operators:
  * `onBackpressureBuffer`
  * `onBackpressureDrop`
* how and when to support producers in custom observables & operators
  * point here from the "how to make a custom operator" page; maybe also from `create` operator doc

_**Work in progress...**_