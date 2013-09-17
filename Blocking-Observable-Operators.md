This section explains the [`BlockingObservable`](http://netflix.github.io/RxJava/javadoc/rx/observables/BlockingObservable.html) subclass. A Blocking Observable extends the ordinary Observable class by providing a set of operators on the items emitted by the Observable that block.

To transform an `Observable` into a `BlockingObservable`, use the [`Observable.toBlockingObservable( )`](http://netflix.github.io/RxJava/javadoc/rx/Observable.html#toBlockingObservable%28%29) method or the [`BlockingObservable.from( )`](http://netflix.github.io/RxJava/javadoc/rx/observables/BlockingObservable.html#from%28rx.Observable%29) method.

* [**`forEach( )`**](Blocking-Observable-Operators#foreach) — invoke a function on each item emitted by the Observable; block until the Observable completes
* [**`last( )`**](Blocking-Observable-Operators#last-and-lastordefault) — block until the Observable completes, then return the last item emitted by the Observable
* [**`lastOrDefault( )`**](Blocking-Observable-Operators#last-and-lastordefault) — block until the Observable completes, then return the last item emitted by the Observable or a default item if there is no last item
* [**`mostRecent( )`**](Blocking-Observable-Operators#mostrecent) — returns an iterable that always returns the item most recently emitted by the Observable
* [**`next( )`**](Blocking-Observable-Operators#next) — returns an iterable that blocks until the Observable emits another item, then returns that item
* [**`single( )`**](Blocking-Observable-Operators#single-and-singleordefault) — if the Observable completes after emitting a single item, return that item, otherwise throw an exception
* [**`singleOrDefault( )`**](Blocking-Observable-Operators#single-and-singleordefault) — if the Observable completes after emitting a single item, return that item, otherwise return a default item
* [**`toFuture( )`**](Blocking-Observable-Operators#transformations-tofuture-toiterable-and-toiteratorgetiterator) — convert the Observable into a Future
* [**`toIterable( )`**](Blocking-Observable-Operators#transformations-tofuture-toiterable-and-toiteratorgetiterator) — convert the sequence emitted by the Observable into an Iterable
* [**`getIterator( )` or `toIterator( )`**](Blocking-Observable-Operators#transformations-tofuture-toiterable-and-toiteratorgetiterator) — convert the sequence emitted by the Observable into an Iterator

> This documentation accompanies its explanations with a modified form of "marble diagrams." Here is how these marble diagrams represent Blocking Observables:

[[images/rx-operators/B.legend.png]]

## forEach( )
#### invoke a function on each item emitted by the Observable; block until the Observable completes
[[images/rx-operators/B.forEach.png]]
The `forEach(someFunction)` method is the blocking equivalent of `subscribe(someFunction)`. When you pass a function to this method, `forEach( )` will invoke your function for each item emitted by the Observable, but will only return control to you once the Observable completes (it will not otherwise indicate that the Observable has completed; there is no `forEach( )` equivalent of the `onError` or `onCompleted` methods).

## last( ) and lastOrDefault( )
#### block until the Observable completes, then return the last item emitted by the Observable
[[images/rx-operators/B.last.png]]
Use the `last( )` method to retrieve the last item emitted by an Observable, at the time the Observable completes (or `null` if the Observable emitted no items before completing).

You can also use this method to retrieve the last item emitted by an Observable that meets some particular condition (or `null` if the Observable method emits no such items). To do this, pass a function to `last( )` that returns `true` if the item meets the condition.
[[images/rx-operators/B.last.p.png]]

Note that because `last( )` emits `null` to indicate that no item (or no matching item) was emitted by the underlying Observable, this creates an ambiguity in the case of Observables whose last emitted item (or matching item) _is_ `null`:

```groovy
def boNull    = Observable.from([null]).toBlockingObservable();
def boNothing = Observable.from([]).toBlockingObservable();
println('boNull.last(): ' + boNull.last());
println('boNothing.last(): ' + boNothing.last());
```
```
boNull.last(): null
boNothing.last(): null
```

The `lastOrDefault( )` method is similar to `last( )`, except that instead of returning `null` when there is no last item (or no last item that meets the specified condition), in such a case it will instead return a default item that you specify. Specify that default item by passing it as the first parameter to `lastOrDefault( )`.
[[images/rx-operators/B.lastOrDefault.png]]
[[images/rx-operators/B.lastOrDefault.p.png]]

Note that you can use this to guard against the ambiguous-`null` noted above:

```groovy
def boNull    = Observable.from([null]).toBlockingObservable();
def boNothing = Observable.from([]).toBlockingObservable();
println('boNull.lastOrDefault("foo"): ' + boNull.lastOrDefault("foo"));
println('boNothing.lastOrDefault("foo"): ' + boNothing.lastOrDefault("foo"));
```
```
boNull.lastOrDefault("foo"): null
boNothing.lastOrDefault("foo"): foo
```

## mostRecent( )
#### returns an iterable that always returns the item most recently emitted by the Observable
[[images/rx-operators/B.mostRecent.png]]
The `mostRecent()` method returns an iterable that on each iteration returns the item that was most recently emitted by the underlying Observable (or `null` if the Observable has not yet emitted an item or has completed without emitting any).

## next( )
#### returns an iterable that blocks until the Observable emits another item, then returns that item
[[images/rx-operators/B.next.png]]
The `next()` method returns an iterable that on each iteration blocks until the underlying Observable emits another item, then returns that item (or `null` if the Observable finishes without emitting another item).

## single( ) and singleOrDefault( )
#### if the Observable completes after emitting a single item, return that item, otherwise throw an exception (or return a default item)
[[images/rx-operators/B.single.png]]
Use the `single( )` method to retrieve the only item emitted by an Observable. `single( )` will throw an exception if the Observable does not emit exactly one item.

You can also use this method to retrieve the only item emitted by an Observable that meets some particular condition (or `null` if the Observable method emits no such item). To do this, pass a function to `single( )` that returns `true` if the item meets the condition. In such a case, `single( )` will again throw an exception unless the Observable emits exactly one item that meets the condition.
[[images/rx-operators/B.single.p.png]]

The `singleOrDefault( )` method is similar, except that while it will still throw an exception if the underlying Observable emits _more than_ one item, if the underlying Observable does not emit any items at all, rather than throwing an exception `singleOrDefault( )` will return a default item that you specify. Specify that default item by passing it as the first parameter to `singleOrDefault( )`.
[[images/rx-operators/B.singleOrDefault.png]]
[[images/rx-operators/B.singleOrDefault.p.png]]

## transformations: toFuture( ), toIterable( ), and toIterator( )/getIterator( )
#### transform an Observable into a Future, an Iterable, or an Iterator
[[images/rx-operators/B.toIterator.png]]
[[images/rx-operators/B.toIterable.png]]
[[images/rx-operators/B.toFuture.png]]

Use these methods to transform a Blocking Observable into a `Future`, an `Iterable`, or an `Iterator`. Note that `toFuture( )` will only work on Blocking Observables that emit one or fewer items. To convert Blocking Observables that emit two or more items into Futures, instead use `.toList( ).toFuture( )` to reduce the items emitted by the Observable to a single (list) item.