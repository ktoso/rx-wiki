This section explains the `BlockingObservable` subclass. A Blocking Observable extends the ordinary Observable class by providing a set of operators on the emissions of the Observable that block.

> To transform an `Observable` into a `BlockingObservable`, use the `Observable.toBlockingObservable( )` method or the `BlockingObservable.from( )` method.

* **`forEach( )`** — invoke a closure on each element emitted by the Observable; block until the Observable completes
* **`last( )`** — block until the Observable completes, then return the last item emitted by the Observable
* **`lastOrDefault( )`** — 
* **`mostRecent( )`** — immediately return the item most recently emitted by the Observable
* **`next( )`** — block until the Observable emits an item, then return that item
* **`single( )`** — if the Observable completes after emitting a single item, return that item, otherwise throw an exception
* **`singleOrDefault( )`** — if the Observable completes after emitting a single item, return that item, otherwise return a default value
* **`toFuture( )`** — convert the Observable into a Future
* **`toIterable( )`** — convert the sequence emitted by the Observable into an Iterable
* **`getIterator( )` or `toIterator( )`** — convert the sequence emitted by the Observable into an Iterator

## forEach( )
#### invoke a closure on each element emitted by the Observable; block until the Observable completes
[[images/rx-operators/B.forEach.png]]

## last( ) and lastOrDefault( )
#### block until the Observable completes, then return the last item emitted by the Observable
[[images/rx-operators/B.last.png]]

## mostRecent( )
#### immediately return the item most recently emitted by the Observable
[[images/rx-operators/B.mostRecent.png]]

## next( )
#### block until the Observable emits an item, then return that item
[[images/rx-operators/B.next.png]]

## single( ) and singleOrDefault( )
#### if the Observable completes after emitting a single item, return that item, otherwise throw an exception (or return a default value)
[[images/rx-operators/B.single.png]]

## transformations: toFuture( ), toIterable( ), and toIterator( )/getIterator( )
#### transform an Observable into a Future, an Iterable, or an Iterator
[[images/rx-operators/B.toIterator.png]]