This section explains various utility operators for working with Observables.

* [**`toList( )`**](Observable-Utility-Operators#tolist) — collect all items from an Observable and emit them as a single List
* [**`toSortedList( )`**](Observable-Utility-Operators#tosortedlist) — collect all items from an Observable and emit them as a single, sorted List
* [**`materialize( )`**](Observable-Utility-Operators#materialize) — convert an Observable into a list of Notifications
* [**`dematerialize( )`**](Observable-Utility-Operators#dematerialize) — convert a materialized Observable back into its non-materialized form
* [**`timestamp( )`**](Observable-Utility-Operators#timestamp) — attach a timestamp to every item emitted by an Observable
* [**`all( )`**](Observable-Utility-Operators#all) — determine whether all items emitted by an Observable meet some criteria
* [**`exists( )` and `isEmpty( )`**](Observable-Utility-Operators#exists-and-isempty) — determine whether an Observable emits any items or not
* [**`contains( )`**](Observable-Utility-Operators#contains) — determine whether an Observable emits a particular item or not
* [**`sequenceEqual( )`**](Observable-Utility-Operators#sequenceequal) — test the equality of pairs of items emitted by two Observables
* [**`synchronize( )`**](Observable-Utility-Operators#synchronize) — force an Observable to make synchronous calls and to be well-behaved
* [**`cache( )`**](Observable-Utility-Operators#cache) — remember the sequence of items emitted by the Observable and emit the same sequence to future Observers
* [**`observeOn( )`**](Observable-Utility-Operators#observeon) — specify on which Scheduler an Observer should observe the Observable
* [**`subscribeOn( )`**](Observable-Utility-Operators#subscribeon) — specify which Scheduler an Observable should use when its subscription is invoked
* [**`parallel( )`**](Observable-Utility-Operators#parallel) — split the work done on the emissions from an Observable into multiple Observables each operating on its own parallel thread
* [**`finallyDo( )`**](Observable-Utility-Operators#finallydo) — register an action to take when an Observable completes
* [**`delay( )`**](Observable-Utility-Operators#delay) — shift the emissions from an Observable forward in time by a specified amount
* [**`timeInterval( )`**](Observable-Utility-Operators#timeinterval) — emit the time lapsed between consecutive emissions of a source Observable
* [**`using( )`**](Observable-Utility-Operators#using) — create a disposable resource that has the same lifespan as an Observable

***

## toList( )
#### collect all items from an Observable and emit them as a single List

[[images/rx-operators/toList.png]]

Normally, an Observable that emits multiple items will do so by invoking its Observer’s `onNext` method for each such item. You can change this behavior, instructing the Observable to compose a list of these multiple items and then to invoke the Observer’s `onNext` method _once_, passing it the entire list, by calling the Observable’s `toList( )` method prior to calling its `subscribe( )` method. For example:

```groovy
Observable.tolist(myObservable).subscribe({ myListOfSomething -> do something useful with the list });
```

For example, the following rather pointless code takes a list of integers, converts it into an Observable, then converts that Observable into one that emits the original list as a single item:

```groovy
numbers = Observable.from([1, 2, 3, 4, 5, 6, 7, 8, 9]);

Observable.toList(numbers).subscribe(
  { println(it); },                  // onNext
  { println("Error encountered"); }, // onError
  { println("Sequence complete"); }  // onCompleted
);
```
```
[1, 2, 3, 4, 5, 6, 7, 8, 9]
Sequence complete
```

In addition to calling `toList( )` as a stand-alone method, you can also call it as a method of an Observable, so, in the example above, instead of 

```groovy
Observable.toList(numbers) ...
```
you could instead write 
```groovy
numbers.toList() ...
```

If you pass to `toList( )` an Observable that invokes `onCompleted` before emitting any items, `toList( )` will emit an empty list before invoking `onCompleted`. If the Observable you pass to `toList( )` invokes `onError`, `toList( )` will in turn invoke the `onError` methods of its Observers.

#### see also
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#toList()">`toList()`</a>
* RxJS: <a href="https://github.com/Reactive-Extensions/RxJS/blob/master/doc/observable.md#rxobservableprototypetoarray">`toArray`</a>
* Linq: <a href="http://msdn.microsoft.com/en-us/library/hh211848.aspx">`ToList`</a>
* Linq: <a href="http://msdn.microsoft.com/en-us/library/hh229207.aspx">`ToArray`</a>

***

## toSortedList( )
#### collect all items emitted by an Observable and emit them as a single sorted List

[[images/rx-operators/toSortedList.png]]

The `toSortedList( )` method behaves much like `toList( )` except that it sorts the resulting list. By default it sorts the list naturally in ascending order by means of the `Comparable` interface. If any of the items emitted by the Observable does not support `Comparable` with respect to the type of every other item emitted by the Observable, `toSortedList( )` will throw an exception. However, you can change this default behavior by also passing in to `toSortedList( )` a function that takes as its parameters two items and returns a number; `toSortedList( )` will then use that function instead of `Comparable` to sort the items.

For example, the following code takes a list of unsorted integers, converts it into an Observable, then converts that Observable into one that emits the original list in sorted form as a single item:

```groovy
numbers = Observable.from([8, 6, 4, 2, 1, 3, 5, 7, 9]);

Observable.toSortedList(numbers).subscribe(
  { println(it); },                  // onNext
  { println("Error encountered"); }, // onError
  { println("Sequence complete"); }  // onCompleted
);
```
```
[1, 2, 3, 4, 5, 6, 7, 8, 9]
Sequence complete
```
Here is an example that provides its own sorting function, in this case, one that sorts numbers according to how close to the number 5 they are:
```groovy
numbers = Observable.from([8, 6, 4, 2, 1, 3, 5, 7, 9]);

Observable.toSortedList(numbers, { n, m -> Math.abs(5-n) - Math.abs(5-m) }).subscribe(
  { println(it); },                  // onNext
  { println("Error encountered"); }, // onError
  { println("Sequence complete"); }  // onCompleted
);
```
```
[5, 6, 4, 3, 7, 8, 2, 1, 9]
Sequence complete
```

In addition to calling `toSortedList( )` as a stand-alone method, you can also call it as a method of an Observable, so, in the examples above, instead of 

```groovy
Observable.toSortedList(numbers) ...
```
or
```groovy
Observable.toSortedList(numbers, { n, m -> Math.abs(5-n) - Math.abs(5-m) }) ...
```
you could instead write
```groovy
numbers.toSortedList() ...
```
or
```groovy
numbers.toSortedList({ n, m -> Math.abs(5-n) - Math.abs(5-m) }) ...
```

#### see also:
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#toSortedList()">`toSortedList()`</a>
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#toSortedList(rx.util.functions.Func2)">`toSortedList(sortingFunction)`</a>

***

## materialize( )
#### convert an Observable into a list of Notifications

[[images/rx-operators/materialize.png]]

A well-formed Observable will invoke its Observer’s `onNext` method zero or more times, and then will invoke either the `onCompleted` or `onError` method exactly once. The `materialize( )` method converts this series of invocations into a series of items emitted by an Observable, where it emits each such invocation as a `Notification` object.

For example:

```groovy
numbers = Observable.from([1, 2, 3]);

Observable.materialize(numbers).subscribe(
  { if(rx.Notification.Kind.OnNext == it.kind) { println("Next: " + it.value); }
    else if(rx.Notification.Kind.OnCompleted == it.kind) { println("Completed"); }
    else if(rx.Notification.Kind.OnError == it.kind) { println("Error: " + it.exception); } },
  { println("Error encountered"); }, // onError
  { println("Sequence complete"); }  // onCompleted
);
```
```
Next: 1
Next: 2
Next: 3
Completed
Sequence complete
```

In addition to calling `materialize( )` as a stand-alone method, you can also call it as a method of an Observable, so that instead of 

```groovy
Observable.materialize(numbers) ...
```
in the above example, you could also write 

```groovy
numbers.materialize() ...
```

#### see also:
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#materialize()">`materialize()`</a>
* Linq: <a href="http://msdn.microsoft.com/en-us/library/hh229453.aspx">`Materialize`</a>
* <a href="http://www.introtorx.com/Content/v1.0.10621.0/08_Transformation.html#MaterializeAndDematerialize">Introduction to Rx: Materialize and Dematerialize</a>

***

## dematerialize( )
#### convert a materialized Observable back into its non-materialized form

[[images/rx-operators/dematerialize.png]]

You can undo the effects of `materialize( )` by means of the `dematerialize( )` method, which will emit the items from the Observable as though `materialize( )` had not been applied to it. The following example dematerializes the materialized Observable from the previous section:
```groovy
numbers = Observable.from([1, 2, 3]);

Observable.materialize(numbers).dematerialize().subscribe(
  { println(it); },                  // onNext
  { println("Error encountered"); }, // onError
  { println("Sequence complete"); }  // onCompleted
);
```
```
1
2
3
Sequence complete
```

#### see also:
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#dematerialize()">`dematerialize()`</a>
* RxJS: <a href="https://github.com/Reactive-Extensions/RxJS/blob/master/doc/observable.md#rxobservableprototypedematerialize">`dematerialize`</a>
* Linq: <a href="http://msdn.microsoft.com/en-us/library/hh229047.aspx">`Dematerialize`</a>
* <a href="http://www.introtorx.com/Content/v1.0.10621.0/08_Transformation.html#MaterializeAndDematerialize">Introduction to Rx: Materialize and Dematerialize</a>

***

## timestamp( )
#### attach a timestamp to every item emitted by an Observable
[[images/rx-operators/timestamp.png]]

The `timestamp( )` method converts an Observable that emits items of type _T_ into one that emits objects of type [`Timestamped<T>`](http://netflix.github.io/RxJava/javadoc/rx/util/Timestamped.html), where each such object is stamped with the time at which it was emitted.

```groovy
def myObservable = Observable.range(1, 1000000).filter({ 0 == (it % 200000) });

myObservable.timestamp().subscribe(
  { println(it.toString()); },       // onNext
  { println("Error encountered"); }, // onError
  { println("Sequence complete"); }  // onCompleted
);
```
```
Timestamped(timestampMillis = 1369252582698, value = 200000)
Timestamped(timestampMillis = 1369252582740, value = 400000)
Timestamped(timestampMillis = 1369252582782, value = 600000)
Timestamped(timestampMillis = 1369252582823, value = 800000)
Timestamped(timestampMillis = 1369252582864, value = 1000000)
Sequence complete
```

#### see also:
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#timestamp()">`timestamp()`</a>
* Linq: <a href="http://msdn.microsoft.com/en-us/library/system.reactive.linq.observable.timestamp.aspx">`Timestamp`</a>
* <a href="http://www.introtorx.com/Content/v1.0.10621.0/08_Transformation.html#TimeStampAndTimeInterval">Introduction to Rx: TimeStamp and TimeInterval</a>

***

## all( )
#### determine whether all items emitted by an Observable meet some criteria
[[images/rx-operators/all.png]]

Pass an function to `all( )` that accepts an item emitted by the source Observable and returns a boolean value based on an evaluation of that item, and `all( )` will emit `true` if and only if that function returned true for every item emitted by the source Observable.

```groovy
numbers = Observable.from([1, 2, 3, 4, 5]);

println("all even?" )
numbers.all({ 0 == (it % 2) }).subscribe({ println(it); });

println("all positive?");
numbers.all({ 0 < it }).subscribe({ println(it); });
```
```
all even? 
false
all positive? 
true
```

#### see also:
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#all(rx.util.functions.Func1)">`all(predicate)`</a>
* RxJS: <a href="https://github.com/Reactive-Extensions/RxJS/blob/master/doc/observable.md#rxobservableprototypeallpredicate-thisarg">`all`</a>
* Linq: <a href="http://msdn.microsoft.com/en-us/library/hh229537.aspx">`All`</a>
* <a href="http://www.introtorx.com/Content/v1.0.10621.0/06_Inspection.html#All">Introduction to Rx: All</a>

***

## exists( ) and isEmpty( )
#### determine whether an Observable emits any items or not
[[images/rx-operators/exists.png]]

When you apply the `exists( )` operator to a source Observable, the resulting Observable will emit `true` and complete if the source Observable emits one or more items before completing, or it will emit `false` and complete if the source Observable completes without emitting any items.

[[images/rx-operators/isEmpty.png]]
The inverse of this is the `isEmpty( )` operator. Apply it to a source Observable and the resulting Observable will emit `true` and complete if the source Observable completes without emitting any items, or it will emit `false` and complete if the source Observable emits any item before completing.

#### see also:
* RxJS: <a href="https://github.com/Reactive-Extensions/RxJS/blob/master/doc/observable.md#rxobservableprototypeanypredicate-thisarg">`any`</a>
* RxJS: <a href="https://github.com/Reactive-Extensions/RxJS/blob/master/doc/observable.md#rxobservableprototypeisempty">`isEmpty`</a>
* Linq: <a href="http://msdn.microsoft.com/en-us/library/system.reactive.linq.observable.any.aspx">`Any`</a>
* <a href="http://www.introtorx.com/Content/v1.0.10621.0/06_Inspection.html#Any">Introduction to Rx: Any</a>

***

## contains( )
#### determine whether an Observable emits a particular item or not
[[images/rx-operators/contains.png]]

Pass the `contains( )` operator a particular item, and it will emit `true` if that item is emitted by the source Observable, or `false` if the source Observable terminates without emitting that item.

#### see also:
* Linq: <a href="http://msdn.microsoft.com/en-us/library/system.reactive.linq.observable.contains.aspx">`Contains`</a>
* <a href="http://www.introtorx.com/Content/v1.0.10621.0/06_Inspection.html#Contains">Introduction to Rx: Contains</a>

***

## sequenceEqual( )
#### test the equality of pairs of items emitted by two Observables
[[images/rx-operators/sequenceEqual.png]]

Pass `sequenceEqual( )` two Observables, and it will compare the items emitted by each Observable, and emit `true` for each pair of items if and only if both items are the same. You can optionally pass a third parameter: a function that accepts two items and returns `true` if they are equal according to a standard of your choosing.
```groovy
def firstfour = Observable.from([1, 2, 3, 4]);
def firstfouragain = Observable.from([1, 2, 3, 4]);
def firstfive = Observable.from([1, 2, 3, 4, 5]);
def firstfourscrambled = Observable.from([3, 2, 1, 4]);

println('firstfour == firstfive?');
Observable.sequenceEqual(firstfour, firstfive).subscribe({ println(it); });
println('firstfour == firstfouragain?');
Observable.sequenceEqual(firstfour, firstfouragain).subscribe({ println(it); });
println('firstfour == firstfourscrambled?');
Observable.sequenceEqual(firstfour, firstfourscrambled).subscribe({ println(it); });
```
```
firstfour == firstfive?
true
true
true
true
firstfour == firstfouragain?
true
true
true
true
firstfour == firstfourscrambled?
false
true
false
true
```

#### see also:
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#sequenceEqual(rx.Observable, rx.Observable)">`sequenceEqual(observable1, observable2)`</a>
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#sequenceEqual(rx.Observable, rx.Observable, rx.util.functions.Func2)">`sequenceEqual(observable1, observable2, equalityFunction)`</a>
* Linq: <a href="http://msdn.microsoft.com/en-us/library/system.reactive.linq.observable.sequenceequal.aspx">`SequenceEqual`</a>
* <a href="http://www.introtorx.com/Content/v1.0.10621.0/06_Inspection.html#SequenceEqual">Introduction to Rx: SequenceEqual</a>

***

## synchronize( )
#### force an Observable to make synchronous calls and to be well-behaved

[[images/rx-operators/synchronize.png]]

It is possible for an Observable to invoke its Observers' methods asynchronously, perhaps in different threads. This could make an Observable poorly-behaved, in that it might invoke `onCompleted` or `onError` before one of its `onNext` invocations. You can force such an Observable to be well-behaved and synchronous by applying the `synchronize( )` method to it.

#### see also:
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#synchronize(rx.Observable)">`synchronize(observable)`</a>
* Linq: <a href="http://msdn.microsoft.com/en-us/library/system.reactive.linq.observable.synchronize.aspx">`Synchronize`</a>

***

## cache( )
#### remember the sequence of items emitted by the Observable and emit the same sequence to future Observers

[[images/rx-operators/cache.png]]

By default, an Observable will generate its sequence of emitted items afresh for each new Observer that subscribes. You can force it to generate its sequence only once and then to emit this identical sequence to every Observer by using the `cache( )` method. Compare the behavior of the following two sets of sample code, the first of which does _not_ use `cache( )` and the second of which does:
```groovy
def myObservable = Observable.range(1, 1000000).filter({ 0 == (it % 400000) }).timestamp();

myObservable.subscribe(
  { println(it.toString()); },       // onNext
  { println("Error encountered"); }, // onError
  { println("Sequence complete"); }  // onCompleted
);
myObservable.subscribe(
  { println(it.toString()); },       // onNext
  { println("Error encountered"); }, // onError
  { println("Sequence complete"); }  // onCompleted
);
```
```
Timestamped(timestampMillis = 1369252832871, value = 400000)
Timestamped(timestampMillis = 1369252832951, value = 800000)
Sequence complete
Timestamped(timestampMillis = 1369252833074, value = 400000)
Timestamped(timestampMillis = 1369252833154, value = 800000)
Sequence complete
```
```groovy
def myObservable = Observable.range(1, 1000000).filter({ 0 == (it % 400000) }).timestamp().cache();

myObservable.subscribe(
  { println(it.toString()); },       // onNext
  { println("Error encountered"); }, // onError
  { println("Sequence complete"); }  // onCompleted
);
myObservable.subscribe(
  { println(it.toString()); },       // onNext
  { println("Error encountered"); }, // onError
  { println("Sequence complete"); }  // onCompleted
);
```
```
Timestamped(timestampMillis = 1369252924548, value = 400000)
Timestamped(timestampMillis = 1369252924630, value = 800000)
Sequence complete
Timestamped(timestampMillis = 1369252924548, value = 400000)
Timestamped(timestampMillis = 1369252924630, value = 800000)
Sequence complete
```
Note that in the second example the timestamps are identical for both of the observers, whereas in the first example they differ.

#### see also:
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#cache()">`cache()`</a>

***

## observeOn( )
#### specify on which Scheduler an Observer should observe the Observable
[[images/rx-operators/observeOn.png]]
To specify in which Scheduler (thread) the Observable should invoke the Observers' `onNext( )`, `onCompleted( )`, and `onError( )` methods, call the Observable's `observeOn( )` method, passing it the appropriate `Scheduler`.

#### see also:
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#observeOn(rx.Scheduler)">`observeOn(scheduler)`</a>
* RxJS: <a href="https://github.com/Reactive-Extensions/RxJS/blob/master/doc/observable.md#rxobservableprototypeobserveonscheduler">`observeOn`</a>
* Linq: <a href="http://msdn.microsoft.com/en-us/library/system.reactive.linq.observable.observeon.aspx">`ObserveOn`</a>
* <a href="http://channel9.msdn.com/Series/Rx-Workshop/Rx-Workshop-Schedulers">Rx Workshop: Schedulers</a>
* <a href="http://www.introtorx.com/Content/v1.0.10621.0/15_SchedulingAndThreading.html#SubscribeOnObserveOn">Introduction to Rx: SubscribeOn and ObserveOn</a>

***

## subscribeOn( )
#### specify which Scheduler an Observable should use when its subscription is invoked
[[images/rx-operators/subscribeOn.png]]

To specify that the work done by the Observable should be done on a particular Scheduler (thread), call the Observable's `subscribeOn( )` method, passing it the appropriate `Scheduler`. By default (that is, unless you modify the Observable also with `observeOn( )`) the Observable will invoke the Observers' `onNext( )`, `onCompleted( )`, and `onError( )` methods in this same thread.

#### see also:
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#subscribeOn(rx.Scheduler)">`subscribeOn(scheduler)`</a>
* RxJS: <a href="https://github.com/Reactive-Extensions/RxJS/blob/master/doc/observable.md#rxobservableprototypesubscribeonscheduler">`subscribeOn`</a>
* Linq: <a href="http://msdn.microsoft.com/en-us/library/system.reactive.linq.observable.subscribeon.aspx">`SubscribeOn`</a>
* <a href="http://channel9.msdn.com/Series/Rx-Workshop/Rx-Workshop-Schedulers">Rx Workshop: Schedulers</a>
* <a href="http://www.introtorx.com/Content/v1.0.10621.0/15_SchedulingAndThreading.html#SubscribeOnObserveOn">Introduction to Rx: SubscribeOn and ObserveOn</a>

***

## parallel( )
#### split the work done on the emissions from an Observable into multiple Observables each operating on its own parallel thread
[[images/rx-operators/parallel.png]]

You can use the `parallel( )` method to split an Observable into as many Observables as there are available processors, and to do work in parallel on each of these Observables. `parallel( )` will then merge the results of these parallel computations back into a single, well-behaved Observable sequence.

***

## finallyDo( )
#### register an action to take when an Observable completes
[[images/rx-operators/finallyDo.png]]

You can use the `finallyDo( )` method of an Observable to register an action (a function that implements `Action0`) that RxJava will invoke when that Observable invokes either the `onCompleted( )` or `onError( )` method of its Observer.

```groovy
class TestFinally
{
  static class myActionClass implements rx.util.functions.Action0 {
    void call() { println('Finally'); }
  }
  
  static main() {
    def myAction = new myActionClass();
    def numbers = Observable.from([1, 2, 3, 4, 5]);
    
    numbers.finallyDo(myAction).subscribe(
      { println(it); },                  // onNext
      { println("Error encountered"); }, // onError
      { println("Sequence complete"); }  // onCompleted
    );
  }
}
new TestFinally().main();
```
```
1
2
3
4
5
Sequence complete
Finally
```

#### see also:
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#finallyDo(rx.util.functions.Action0)">`finallyDo(action)`</a>
* RxJS: <a href="https://github.com/Reactive-Extensions/RxJS/blob/master/doc/observable.md#rxobservableprototypefinallyaction">`finally` / `finallyAction`</a>
* Linq: <a href="http://msdn.microsoft.com/en-us/library/hh212133.aspx">`Finally`</a>

***

## delay( )
#### shift the emissions from an Observable forward in time by a specified amount
[[images/rx-operators/delay.png]]

The `delay( )` operator modifies its source Observable by pausing for a particular increment of time (that you specify) before emitting each of the source Observable's items. This has the effect of shifting the entire sequence of items emitted by the Observable forward in time by that specified increment.

Note that `delay( )` will _not_ time-shift an `onError( )` call in this fashion but it will forward such a call immediately to its subscribers.

#### see also:
* RxJS: <a href="https://github.com/Reactive-Extensions/RxJS/blob/master/doc/observable.md#rxobservableprototypedelayduetime-scheduler">`delay`</a>
* Linq: <a href="http://msdn.microsoft.com/en-us/library/system.reactive.linq.observable.delay.aspx">`Delay`</a>
* <a href="http://www.introtorx.com/Content/v1.0.10621.0/13_TimeShiftedSequences.html#Delay">Introduction to Rx: Delay</a>

***

## timeInterval( )
#### emit the time lapsed between consecutive emissions of a source Observable
[[images/rx-operators/timeInterval.png]]

The `timeInterval( )` operator converts a source Observable into an Observable that emits the amount of time lapsed between consecutive emissions of the source Observable. The first emission is the amount of time lapsed between the time the Observer subscribed to the Observable and the time the source Observable emitted its first item. There is no corresponding emission marking the amount of time lapsed between the last emission of the source Observable and the subsequent call to `onCompleted( )`.

#### see also:
* <a href="http://www.introtorx.com/Content/v1.0.10621.0/08_Transformation.html#TimeStampAndTimeInterval">Introduction to Rx: TimeStamp and TimeInterval</a>
* RxJS: <a href="https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md#rxobservableprototypetimeintervalscheduler">`timeInterval`</a>
* Linq: <a href="http://msdn.microsoft.com/en-us/library/system.reactive.linq.observable.timeinterval.aspx">`TimeInterval`</a>

***

## using( )
#### create a disposable resource that has the same lifespan as an Observable
[[images/rx-operators/using.png]]

Pass the `using( )` method two factory functions: the first creates a disposable resource, the second creates an Observable. When an observer subscribes to the resulting Observable, `using( )` will use the Observable factory function to create the Observable the observer will observe, while at the same time using the resource factory function to create a resource. When the Observer unsubscribes from the Observable, or when the Observable terminates (normally or with an error), `using( )` will dispose of the resource it created.