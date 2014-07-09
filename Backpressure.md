_**Work in progress...**_

In RxJava it is not difficult to get into a situation in which an Observable is emitting items more rapidly than an operator or subscriber can consume them. This presents the problem of what to do with such a growing backlog of unconsumed items.

For example, imagine using the [`zip`](Combining-Observables#wiki-zip) operator to zip together two infinite Observables, one of which emits items twice as frequently as the other. The `zip` operator, to perform as advertised, will have to maintain an ever-expanding buffer of items emitted by the faster Observable to combine with items emitted by the slower one. This could cause RxJava to seize an unwieldy amount of system resources.

You can tell RxJava how you want it to handle cases like these. RxJava is capable of exerting _backpressure_ on Observables. This page tells you how the backpressure options work, and also how you can design your own Observables and Observable operators to respect backpressure requests.

## Useful Operators for Avoiding the Need for Backpressure

Your first line of defense against the problems of over-producing Observables is the ordinary set of Observable operators. In particular, operators like [`sample( )` or `throttleLast( )`](Filtering-Observables#wiki-sample-or-throttlelast), [`throttleFirst( )`](Filtering-Observables#wiki-throttlefirst), and [`throttleWithTimeout( )` or `debounce( )`](Filtering-Observables#wiki-throttlewithtimeout-or-debounce) allow you to regulate the rate at which an Observable emits items.

We might, for example, have used one of these operators on each of the two Observables we intended to `zip` together in the conundrum mentioned earlier, and this would have solved our problem.  But the behavior of the resulting `zip` would also have been different. It would no longer necessarily zip together the <i>n</i>th from each Observable sequentially.

Backpressure allows us to maintain the ordinary behavior of an operator like `zip` until and unless the buffer of unconsumed items grows too large.

_**Work in progress...**_

Things that may need explaining:
* the `Producer` interface (and its `request` method, and how to request w/o backpressure)
* the new methods in `Subscriber`
  * `request(n)`
  * `setProducer(p)`
  * `onStart()`
  * and the new constructors that take a `bufferRequest` parameter
* the new `Observable` operators:
  * `onBackpressureBuffer`
  * `onBackpressureDrop`
* how and when to support producers in custom observables & operators
  * point here from the "how to make a custom operator" page; maybe also from `create` operator doc

_**Work in progress...**_