This section explains various utility operators for working with Observables.

* [**`toList( )`**](Observable-Utility-Operators#tolist) — collect all items from an Observable and emit them as a single List
* [**`toSortedList( )`**](Observable-Utility-Operators#tosortedlist) — collect all items from an Observable and emit them as a single, sorted List
* [**`materialize( )`**](Observable-Utility-Operators#materialize) — convert an Observable into a list of Notifications
* [**`dematerialize( )`**](Observable-Utility-Operators#dematerialize) — convert a materialized Observable back into its non-materialized form
* [**`timestamp( )`**](Observable-Utility-Operators#timestamp) — attach a timestamp to every item emitted by an Observable
* [**`all( )`**](Observable-Utility-Operators#all) — determine whether all items emitted by an Observable meet some criteria
* [**`sequenceEqual( )`**](Observable-Utility-Operators#sequenceequal) — test the equality of pairs of items emitted by two Observables
* [**`synchronize( )`**](Observable-Utility-Operators#synchronize) — force an Observable to make synchronous calls and to be well-behaved
* [**`cache( )`**](Observable-Utility-Operators#cache) — remember the sequence of items emitted by the Observable and emit the same sequence to future Observers
* [**`observeOn( )`**](Observable-Utility-Operators#observeon) — specify on which Scheduler an Observer should observe the Observable
* [**`subscribeOn( )`**](Observable-Utility-Operators#subscribeon) — specify which Scheduler an Observable should use when its subscription is invoked
* [**`onErrorResumeNext( )`**](Observable-Utility-Operators#onerrorresumenext) — instructs an Observable to continue emitting items after it encounters an error
* [**`onErrorReturn( )`**](Observable-Utility-Operators#onerrorreturn) — instructs an Observable to emit a particular item when it encounters an error
* [**`onExceptionResumeNextViaObservable( )`**](Observable-Utility-Operators#onexceptionresumenextviaobservable) — instructs an Observable to continue emitting items after it encounters an exception (but not another variety of throwable)
* [**`finallyDo( )`**](Observable-Utility-Operators#finallydo) — register an action to take when an Observable completes

## toList( )
#### collect all items from an Observable and emit them as a single List

[[images/rx-operators/toList.png]]

Normally, an Observable that emits multiple items will do so by invoking its Observer’s `onNext` method for each such item. You can change this behavior, instructing the Observable to compose a list of these multiple items and then to invoke the Observer’s `onNext` method _once_, passing it the entire list, by calling the Observable’s `toList( )` method prior to calling its `subscribe( )` method. For example:

```groovy
Observable.tolist(myObservable).subscribe([ onNext: { myListOfSomething -> do something useful with the list } ]);
```

For example, the following rather pointless code takes a list of integers, converts it into an Observable, then converts that Observable into one that emits the original list as a single item:

```groovy
numbers = Observable.toObservable([1, 2, 3, 4, 5, 6, 7, 8, 9]);

Observable.toList(numbers).subscribe(
  [ onNext:{ myWriter.println(it); },
    onCompleted:{ myWriter.println("Sequence complete"); },
    onError:{ myWriter.println("Error encountered"); } ]
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

## toSortedList( )
#### collect all items emitted by an Observable and emit them as a single sorted List

[[images/rx-operators/toSortedList.png]]

The `toSortedList( )` method behaves much like `toList( )` except that it sorts the resulting list. By default it sorts the list naturally in ascending order by means of the `Comparable` interface. If any of the items emitted by the Observable does not support `Comparable` with respect to the type of every other item emitted by the Observable, `toSortedList( )` will throw an exception. However, you can change this default behavior by also passing in to `toSortedList( )` a function that takes as its parameters two items and returns a number; `toSortedList( )` will then use that function instead of `Comparable` to sort the items.

For example, the following code takes a list of unsorted integers, converts it into an Observable, then converts that Observable into one that emits the original list in sorted form as a single item:

```groovy
numbers = Observable.toObservable([8, 6, 4, 2, 1, 3, 5, 7, 9]);

Observable.toSortedList(numbers).subscribe(
  [ onNext:{ myWriter.println(it); },
    onCompleted:{ myWriter.println("Sequence complete"); },
    onError:{ myWriter.println("Error encountered"); } ]
);
```
```
[1, 2, 3, 4, 5, 6, 7, 8, 9]
Sequence complete
```
Here is an example that provides its own sorting function, in this case, one that sorts numbers according to how close to the number 5 they are:
```groovy
numbers = Observable.toObservable([8, 6, 4, 2, 1, 3, 5, 7, 9]);

Observable.toSortedList(numbers, { n, m -> Math.abs(5-n) - Math.abs(5-m) }).subscribe(
  [ onNext:{ myWriter.println(it); },
    onCompleted:{ myWriter.println("Sequence complete"); },
    onError:{ myWriter.println("Error encountered"); } ]
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

## materialize( )
#### convert an Observable into a list of Notifications

[[images/rx-operators/materialize.png]]

A well-formed Observable will invoke its Observer’s `onNext` method zero or more times, and then will invoke either the `onCompleted` or `onError` method exactly once. The `materialize( )` method converts this series of invocations into a series of items emitted by an Observable, where it emits each such invocation as a `Notification` object.

For example:

```groovy
numbers = Observable.toObservable([1, 2, 3]);

Observable.materialize(numbers).subscribe(
  [ onNext: { if(rx.Notification.Kind.OnNext == it.kind) { myWriter.println("Next: " + it.value); }
              else if(rx.Notification.Kind.OnCompleted == it.kind) { myWriter.println("Completed"); }
              else if(rx.Notification.Kind.OnError == it.kind) { myWriter.println("Error: " + it.exception); } },
    onCompleted:{ myWriter.println("Sequence complete"); },
    onError:{ myWriter.println("Error encountered"); } ]
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

## dematerialize( )
#### convert a materialized Observable back into its non-materialized form

[[images/rx-operators/dematerialize.png]]

You can undo the effects of `materialize( )` by means of the `dematerialize( )` method, which will emit the items from the Observable as though `materialize( )` had not been applied to it. The following example dematerializes the materialized Observable from the previous section:
```groovy
numbers = Observable.toObservable([1, 2, 3]);

Observable.materialize(numbers).dematerialize().subscribe(
  [ onNext: { myWriter.println(it); },
    onCompleted:{ myWriter.println("Sequence complete"); },
    onError:{ myWriter.println("Error encountered"); } ]
);
```
```
1
2
3
Sequence complete
```

## timestamp( )
#### attach a timestamp to every item emitted by an Observable
[[images/rx-operators/timestamp.png]]

The `timestamp( )` method converts an Observable that emits items of type _T_ into one that emits objects of type [`Timestamped<T>`](http://netflix.github.io/RxJava/javadoc/rx/util/Timestamped.html), where each such object is stamped with the time at which it was emitted.

```groovy
def myObservable = Observable.range(1, 1000000).filter({ 0 == (it % 200000) });

myObservable.timestamp().subscribe(
  [ onNext: { myWriter.println(it.toString()); },
    onCompleted:{ myWriter.println("Sequence complete"); },
    onError:{ myWriter.println("Error encountered"); } ]
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

## all( )
#### determine whether all items emitted by an Observable meet some criteria

[[images/rx-operators/all.png]]

Pass an function to `all( )` that accepts an item emitted by the source Observable and returns a boolean value based on an evaluation of that item, and `all( )` will emit `true` if and only if that function returned true for every item emitted by the source Observable.

```groovy
numbers = Observable.toObservable([1, 2, 3, 4, 5]);

myWriter.println("all even?" )
numbers.all({ 0 == (it % 2) }).subscribe([onNext:{ myWriter.println(it); }]);

myWriter.println("all positive?");
numbers.all({ 0 < it }).subscribe([onNext:{ myWriter.println(it); }]);
```
```
all even? 
false
all positive? 
true
```

## sequenceEqual( )
#### test the equality of pairs of items emitted by two Observables

[[images/rx-operators/sequenceEqual.png]]

Pass `sequenceEqual( )` two Observables, and it will compare the items emitted by each Observable, and emit `true` for each pair of items if and only if both items are the same. You can optionally pass a third parameter: a function that accepts two items and returns `true` if they are equal according to a standard of your choosing.
```groovy
def firstfour = Observable.toObservable([1, 2, 3, 4]);
def firstfouragain = Observable.toObservable([1, 2, 3, 4]);
def firstfive = Observable.toObservable([1, 2, 3, 4, 5]);
def firstfourscrambled = Observable.toObservable([3, 2, 1, 4]);

myWriter.println('firstfour == firstfive?');
Observable.sequenceEqual(firstfour, firstfive).subscribe([onNext:{ myWriter.println(it); }]);
myWriter.println('firstfour == firstfouragain?');
Observable.sequenceEqual(firstfour, firstfouragain).subscribe([onNext:{ myWriter.println(it); }]);
myWriter.println('firstfour == firstfourscrambled?');
Observable.sequenceEqual(firstfour, firstfourscrambled).subscribe([onNext:{ myWriter.println(it); }]);
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

## synchronize( )
#### force an Observable to make synchronous calls and to be well-behaved

[[images/rx-operators/synchronize.png]]

It is possible for an Observable to invoke its Observers' methods asynchronously, perhaps in different threads. This could make an Observable poorly-behaved, in that it might invoke `onCompleted` or `onError` before one of its `onNext` invocations. You can force such an Observable to be well-behaved and synchronous by applying the `synchronize( )` method to it.

## cache( )
#### remember the sequence of items emitted by the Observable and emit the same sequence to future Observers

[[images/rx-operators/cache.png]]

By default, an Observable will generate its sequence of emitted items afresh for each new Observer that subscribes. You can force it to generate its sequence only once and then to emit this identical sequence to every Observer by using the `cache( )` method. Compare the behavior of the following two sets of sample code, the first of which does _not_ use `cache( )` and the second of which does:
```groovy
def myObservable = Observable.range(1, 1000000).filter({ 0 == (it % 400000) }).timestamp();

myObservable.subscribe(
  [ onNext: { myWriter.println(it.toString()); },
    onCompleted:{ myWriter.println("Sequence complete"); },
    onError:{ myWriter.println("Error encountered"); } ]
);
myObservable.subscribe(
  [ onNext: { myWriter.println(it.toString()); },
    onCompleted:{ myWriter.println("Sequence complete"); },
    onError:{ myWriter.println("Error encountered"); } ]
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
  [ onNext: { myWriter.println(it.toString()); },
    onCompleted:{ myWriter.println("Sequence complete"); },
    onError:{ myWriter.println("Error encountered"); } ]
);
myObservable.subscribe(
  [ onNext: { myWriter.println(it.toString()); },
    onCompleted:{ myWriter.println("Sequence complete"); },
    onError:{ myWriter.println("Error encountered"); } ]
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

## observeOn( )
#### specify on which Scheduler an Observer should observe the Observable
[[images/rx-operators/observeOn.png]]
To specify in which Scheduler (thread) the Observable should invoke the Observers' `onNext( )`, `onCompleted( )`, and `onError( )` methods, call the Observable's `observeOn( )` method, passing it the appropriate `Scheduler`.

## subscribeOn( )
#### specify which Scheduler an Observable should use when its subscription is invoked
[[images/rx-operators/subscribeOn.png]]

To specify that the work done by the Observable should be done on a particular Scheduler (thread), call the Observable's `subscribeOn( )` method, passing it the appropriate `Scheduler`. By default (that is, unless you modify the Observable also with `observeOn( )`) the Observable will invoke the Observers' `onNext( )`, `onCompleted( )`, and `onError( )` methods in this same thread.

## onErrorResumeNext( )
#### instructs an Observable to attempt to continue emitting items after it encounters an error
[[images/rx-operators/onErrorResumeNext.png]]

The `onErrorResumeNext( )` method returns an Observable that mirrors the behavior of the source Observable, _unless_ that Observable invokes `onError( )` in which case, rather than propagating that error to the Observer, `onErrorResumeNext( )` will instead begin mirroring a second, backup Observable, as shown in the following sample code:
```groovy
def myObservable = Observable.create({ anObserver ->
  anObserver.onNext('Three');
  anObserver.onNext('Two');
  anObserver.onNext('One');
  anObserver.onError();
});
def myFallback = Observable.create({ anObserver ->
  anObserver.onNext('0');
  anObserver.onNext('1');
  anObserver.onNext('2');
  anObserver.onCompleted();
});

myObservable.onErrorResumeNext(myFallback).subscribe(
  [ onNext:{ myWriter.println(it); },
    onCompleted:{ myWriter.println("Sequence complete"); },
    onError:{ myWriter.println("Error encountered"); } ]
);
```
```
Three
Two
One
0
1
2
Sequence complete
```

## onErrorReturn( )
#### instructs an Observable to emit a particular item to an Observer’s onNext method when it encounters an error
[[images/rx-operators/onErrorReturn.png]]

The `onErrorReturn( )` method returns an Observable that mirrors the behavior of the source Observable, _unless_ that Observable invokes `onError( )` in which case, rather than propagating that error to the Observer, `onErrorReturn( )` will instead emit a specified item and invoke the Observer's `onCompleted( )` method, as shown in the following sample code:
```groovy
def myObservable = Observable.create({ anObserver ->
  anObserver.onNext('Four');
  anObserver.onNext('Three');
  anObserver.onNext('Two');
  anObserver.onNext('One');
  anObserver.onError();
});

myObservable.onErrorReturn({ return('Blastoff!'); }).subscribe(
  [ onNext:{ myWriter.println(it); },
    onCompleted:{ myWriter.println("Sequence complete"); },
    onError:{ myWriter.println("Error encountered"); } ]
);
```
```
Four
Three
Two
One
Blastoff!
Sequence complete
```

## onExceptionResumeNextViaObservable( )
#### instructs an Observable to continue emitting items after it encounters an exception (but not another variety of throwable)
[[images/rx-operators/onExceptionResumeNextViaObservable.png]]

Much like `onErrorResumeNext( )` method, this returns an Observable that mirrors the behavior of the source Observable, _unless_ that Observable invokes `onError( )` in which case, if the `Throwable` passed to `onError( )` is an `Exception`, rather than propagating that error to the Observer, `onErrorResumeNext( )` will instead begin mirroring a second, backup Observable. If the `Throwable` is not an `Exception`, the Observable returned by `onErrorResumeNext( )` will propagate it to its observers' `onError( )` method and will not invoke its backup Observable.

## finallyDo( )
#### register an action to take when an Observable completes

[[images/rx-operators/finallyDo.png]]

You can use the `finallyDo( )` method of an Observable to register an action (a function that implements `Action0`) that RxJava will invoke when that Observable invokes either the `onCompleted( )` or `onError( )` method of its Observer.

```groovy
class TestFinally
{
  static class myActionClass implements rx.util.functions.Action0 {
    void call() { myWriter.println('Finally'); myWriter.flush(); }
  }
  
  static main() {
    def myAction = new myActionClass();
    def numbers = Observable.toObservable([1, 2, 3, 4, 5]);
    
    numbers.finallyDo(myAction).subscribe(
          [ onNext: { myWriter.println(it); },
            onCompleted:{ myWriter.println("Sequence complete"); },
            onError:{ myWriter.println("Error encountered"); } ]
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