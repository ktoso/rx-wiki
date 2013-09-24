This section explains methods that create Observables.

* [**`from( )`**](Creating-Observables#from) — convert an Iterable or a Future into an Observable
* [**`just( )`**](Creating-Observables#just) — convert an object into an Observable that emits that object
* [**`create( )`**](Creating-Observables#create) — create an Observable from scratch by means of a function
* [**`defer( )`**](Creating-Observables#defer) — do not create the Observable until an Observer subscribes; create a fresh Observable on each subscription
* [**`range( )`**](Creating-Observables#range) — create an Observable that emits a range of sequential integers
* [**`interval( )`**](Creating-Observables#interval) — create an Observable that emits a sequence of integers spaced by a given time interval
* [**`empty( )`**](Creating-Observables#empty-error-and-never) — create an Observable that emits nothing and then completes
* [**`error( )`**](Creating-Observables#empty-error-and-never) — create an Observable that emits nothing and then signals an error
* [**`never( )`**](Creating-Observables#empty-error-and-never) — create an Observable that emits nothing at all

***

## from( )
#### convert an Iterable or a Future into an Observable
[[images/rx-operators/from.png]]

You can convert an object that supports `Iterable` into an Observable that emits each iterable item in the object, or an object that supports `Future` into an Observable that emits the result of the `get` call, simply by passing the object into the `from( )` methods, for example:

```groovy
myObservable = Observable.from(myIterable);
```

You can also do this with arrays, for example:

```groovy
myArray = [1, 2, 3, 4, 5];
myArrayObservable = Observable.from(myArray);
```

This converts the sequence of values in the iterable object or array into a sequence of items emitted, one at a time, by an Observable.

An empty iterable (or array) can be converted to an Observable in this way. The resulting Observable will invoke `onCompleted()` without first invoking `onNext()`.

Note that when the `from( )` method transforms a `Future` into an Observable, such an Observable will be effectively blocking, as its underlying `Future` blocks.

#### see also:
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#from(java.util.concurrent.Future)">`from(future)`</a>
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#from(java.util.concurrent.Future, long, java.util.concurrent.TimeUnit)">`from(future, timeout, unit)`</a>
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#from(java.util.concurrent.Future, rx.Scheduler)">`from(future, scheduler)`</a>
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#from(java.lang.Iterable)">`from(iterable)`</a>
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#from(T...)">`from(array)`</a>
* RxJS: [`fromArray`](https://github.com/Reactive-Extensions/RxJS/wiki/Observable#wiki-fromArray)
* Linq: [`ToObservable`](http://msdn.microsoft.com/en-us/library/system.reactive.linq.observable.toobservable%28v=vs.103%29.aspx)

***

## just( )
#### convert an object into an Observable that emits that object
[[images/rx-operators/just.png]]

To convert any object into an Observable that emits that object, pass that object into the `just( )` method.

```groovy
// Observable emits "some string" as a single item
def observableThatEmitsAString = Observable.just("some string"); 
// Observable emits the list [1, 2, 3, 4, 5] as a single item
def observableThatEmitsAList = Observable.just([1, 2, 3, 4, 5]); 
```

This has some similarities to the `from( )` method, but note that if you pass an iterable to `from( )`, it will convert an iterable object into an Observable that emits each of the items in the iterable, one at a time, while the `just( )` method would convert the iterable into an Observable that emits the entire iterable as a single item.

If you pass nothing or `null` to `just( )`, the resulting Observable will _not_ merely call `onCompleted( )` without calling `onNext( )`. It will instead call `onNext( null )` before calling `onCompleted( )`.

#### see also:
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#just(T)">`just(value)`</a>

***

## create( )
#### create an Observable from scratch by means of a function
[[images/rx-operators/create.png]]

You can create an Observable from scratch by using the `create( )` method. You pass this method a function that accepts as its parameter the Observer that is passed to an Observable’s `subscribe( )` method. Write the function you pass to `create( )` so that it behaves as an Observable — calling the passed-in Observer’s `onNext( )`, `onError( )`, and `onCompleted( )` methods appropriately. For example:

```groovy
def myObservable = Observable.create({ anObserver ->
  anObserver.onNext('One');
  anObserver.onNext('Two');
  anObserver.onNext('Three');
  anObserver.onNext('Four');
  anObserver.onCompleted();
})
```

**NOTE:** A well-formed Observable _must_ call either the observer’s `onCompleted( )` method exactly once or its `onError( )` method exactly once, and must not thereafter call any of the observer’s other methods.

#### see also:
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#create(rx.Observable.OnSubscribeFunc)">`create(func)`</a>
* RxJS: [`create`](https://github.com/Reactive-Extensions/RxJS/wiki/Observable#wiki-create)
* Linq: [`Create`](http://msdn.microsoft.com/en-us/library/system.reactive.linq.observable.create%28v=vs.103%29.aspx)

***

## defer( )
#### do not create the Observable until an Observer subscribes; create a fresh Observable on each subscription
[[images/rx-operators/defer.png]]

Pass `defer( )` an Observable factory function (a function that generates Observables), and `defer( )` will return an Observable that will call this function to generate its Observable sequence afresh each time a new Observer subscribes.

#### see also:
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#defer(rx.util.functions.Func0)">`defer(observableFactory)`</a>
* RxJS: [`defer`](https://github.com/Reactive-Extensions/RxJS/wiki/Observable#wiki-defer)
* Linq: [`Defer`](http://msdn.microsoft.com/en-us/library/hh229160%28v=vs.103%29.aspx)

***

## range( )
#### create an Observable that emits a range of sequential integers
[[images/rx-operators/range.png]]

To create an Observable that emits a range of sequential integers, pass the starting integer and the number of integers to emit to the `range( )` method.
```groovy
// myObservable emits the integers 5, 6, and 7 before completing:
def myObservable = Observable.range(5, 3);
```

In calls to `range(n,m)`, values less than 1 for _m_ will result in no numbers being emitted. _n_ may be any integer that can be represented as a `BigDecimal` — posititve, negative, or zero.

#### see also:
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#range(int, int)">`range(start, count)`</a>
* RxJS: [`range`](https://github.com/Reactive-Extensions/RxJS/wiki/Observable#wiki-range)
* Linq: [`Range`](http://msdn.microsoft.com/en-us/library/system.reactive.linq.observable.range%28v=vs.103%29.aspx)

***

## interval( )
#### create an Observable that emits a sequence of integers spaced by a given time interval
[[images/rx-operators/interval.png]]

To create an Observable that emits items spaced by a particular interval of time, pass the time interval and the units of time that interval is measured in (and, optionally, a scheduler) to the `interval( )` method.

#### see also:
* RxJS: [`interval`](https://github.com/Reactive-Extensions/RxJS/wiki/Observable#wiki-interval)
* Linq: [`Interval`](http://msdn.microsoft.com/en-us/library/system.reactive.linq.observable.interval%28v=vs.103%29.aspx)

***

## empty( ), error( ), and never( )
#### Observables that can be useful for testing purposes

* `empty( )` creates an Observable that does not emit any items but instead immediately calls the observer’s `onCompleted( )` method.
[[images/rx-operators/empty.png]]
* `error( )` creates an Observable that does not emit any items but instead immediately calls the observer’s `onError( )` method.
[[images/rx-operators/error.png]]
* `never( )` creates an Observable that does not emit any items, nor does it call either the observer’s `onCompleted( )` or `onError( )` methods.
[[images/rx-operators/never.png]]

```groovy
println("*** empty() ***");
Observable.empty().subscribe(
  { println("empty: " + it); },             // onNext
  { println("empty: Error encountered"); }, // onError
  { println("empty: Sequence complete"); }  // onCompleted
);

println("*** error() ***");
Observable.error().subscribe(
  { println("error: " + it); },             // onNext
  { println("error: Error encountered"); }, // onError
  { println("error: Sequence complete"); }  // onCompleted
);

println("*** never() ***");
Observable.never().subscribe(
  { println("never: " + it); },             // onNext
  { println("never: Error encountered"); }, // onError
  { println("never: Sequence complete"); }  // onCompleted
);
println("*** END ***");
```
```
*** empty() ***
empty: Sequence complete
*** error() ***
error: Error encountered
*** never() ***
*** END ***
```