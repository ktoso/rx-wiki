This section explains various utility operators for working with Observables.

* [**`toList( )`**](Observable-Utility-Operators#tolist) — collect all items from an Observable and emit them as a single List
* [**`toSortedList( )`**](Observable-Utility-Operators#tosortedlist) — collect all items from an Observable and emit them as a single, sorted List
* [**`toMap( )`**](Observable-Utility-Operators#tomap-and-tomultimap) — convert the sequence of items emitted by an Observable into a map keyed by a specified key function
* [**`toMultiMap( )`**](Observable-Utility-Operators#tomap-and-tomultimap) — convert the sequence of items emitted by an Observable into an ArrayList that is also a map keyed by a specified key function
* [**`materialize( )`**](Observable-Utility-Operators#materialize) — convert an Observable into a list of Notifications
* [**`dematerialize( )`**](Observable-Utility-Operators#dematerialize) — convert a materialized Observable back into its non-materialized form
* [**`timestamp( )`**](Observable-Utility-Operators#timestamp) — attach a timestamp to every item emitted by an Observable
* [**`synchronize( )`**](Observable-Utility-Operators#synchronize) — force an Observable to make synchronous calls and to be well-behaved
* [**`cache( )`**](Observable-Utility-Operators#cache) — remember the sequence of items emitted by the Observable and emit the same sequence to future Observers
* [**`observeOn( )`**](Observable-Utility-Operators#observeon) — specify on which Scheduler an Observer should observe the Observable
* [**`subscribeOn( )`**](Observable-Utility-Operators#subscribeon) — specify which Scheduler an Observable should use when its subscription is invoked
* [**`parallel( )`**](Observable-Utility-Operators#parallel) — split the work done on the emissions from an Observable into multiple Observables each operating on its own parallel thread
* [**`doOnEach( )`**](Observable-Utility-Operators#dooneach) — register an action to take whenever an Observable emits an item
* [**`doOnCompleted( )`**](Observable-Utility-Operators#dooncompleted) — register an action to take when an Observable completes successfully
* [**`doOnError( )`**](Observable-Utility-Operators#doonerror) — register an action to take when an Observable completes with an error
* [**`finallyDo( )`**](Observable-Utility-Operators#finallydo) — register an action to take when an Observable completes
* [**`delay( )`**](Observable-Utility-Operators#delay) — shift the emissions from an Observable forward in time by a specified amount
* [**`delaySubscription( )`**](Observable-Utility-Operators#delaysubscription) — hold an Observer's subscription request for a specified amount of time before passing it on to the source Observable
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

numbers.toList().subscribe(
  { println(it); },                          // onNext
  { println("Error: " + it.getMessage()); }, // onError
  { println("Sequence complete"); }          // onCompleted
);
```
```
[1, 2, 3, 4, 5, 6, 7, 8, 9]
Sequence complete
```

If the source Observable invokes `onCompleted` before emitting any items, `toList( )` will emit an empty list before invoking `onCompleted`. If the source Observable invokes `onError`, `toList( )` will in turn invoke the `onError` methods of its Observers.

#### see also
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#toList()">`toList()`</a>
* RxJS: <a href="https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md#rxobservableprototypetoarray">`toArray`</a>
* Linq: <a href="http://msdn.microsoft.com/en-us/library/hh211848.aspx">`ToList`</a> and <a href="http://msdn.microsoft.com/en-us/library/hh229207.aspx">`ToArray`</a>

***

## toSortedList( )
#### collect all items emitted by an Observable and emit them as a single sorted List

[[images/rx-operators/toSortedList.png]]

The `toSortedList( )` method behaves much like `toList( )` except that it sorts the resulting list. By default it sorts the list naturally in ascending order by means of the `Comparable` interface. If any of the items emitted by the Observable does not support `Comparable` with respect to the type of every other item emitted by the Observable, `toSortedList( )` will throw an exception. However, you can change this default behavior by also passing in to `toSortedList( )` a function that takes as its parameters two items and returns a number; `toSortedList( )` will then use that function instead of `Comparable` to sort the items.

For example, the following code takes a list of unsorted integers, converts it into an Observable, then converts that Observable into one that emits the original list in sorted form as a single item:

```groovy
numbers = Observable.from([8, 6, 4, 2, 1, 3, 5, 7, 9]);

numbers.toSortedList().subscribe(
  { println(it); },                          // onNext
  { println("Error: " + it.getMessage()); }, // onError
  { println("Sequence complete"); }          // onCompleted
);
```
```
[1, 2, 3, 4, 5, 6, 7, 8, 9]
Sequence complete
```
Here is an example that provides its own sorting function, in this case, one that sorts numbers according to how close to the number 5 they are:
```groovy
numbers = Observable.from([8, 6, 4, 2, 1, 3, 5, 7, 9]);

numbers.toSortedList({ n, m -> Math.abs(5-n) - Math.abs(5-m) }).subscribe(
  { println(it); },                          // onNext
  { println("Error: " + it.getMessage()); }, // onError
  { println("Sequence complete"); }          // onCompleted
);
```
```
[5, 6, 4, 3, 7, 8, 2, 1, 9]
Sequence complete
```

#### see also:
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#toSortedList()">`toSortedList()`</a>
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#toSortedList(rx.util.functions.Func2)">`toSortedList(sortingFunction)`</a>

***

# toMap( ) and toMultiMap( )
#### convert the sequence of items emitted by an Observable into a map keyed by a specified key function
[[images/rx-operators/toMap.png]]

The `toMap( )` and `toMultiMap( )` methods collect the items emitted by the source Observable into a map (by default, a `HashMap`, but you can supply a factory function that generates another `Map` variety) and then emit that map. You supply a function that generates the key for each emitted item. You may also optionally supply a function that converts an emitted item into the value to be stored in the map (by default, the item itself is this value).

The `toMultiMap( )` method differs from `toMap( )` in that the map it generates is also an `ArrayList`.
[[images/rx-operators/toMultiMap.png]]

#### see also:
* Linq: <a href="http://msdn.microsoft.com/en-us/library/system.reactive.linq.observable.tolookup.aspx">`ToLookup`</a> and <a href="http://msdn.microsoft.com/en-us/library/system.reactive.linq.observable.todictionary.aspx">`ToDictionary`</a>

***

## materialize( )
#### convert an Observable into a list of Notifications
[[images/rx-operators/materialize.png]]

A well-formed Observable will invoke its Observer’s `onNext` method zero or more times, and then will invoke either the `onCompleted` or `onError` method exactly once. The `materialize( )` method converts this series of invocations into a series of items emitted by an Observable, where it emits each such invocation as a `Notification` object.

For example:

```groovy
numbers = Observable.from([1, 2, 3]);

numbers.materialize().subscribe(
  { if(rx.Notification.Kind.OnNext == it.kind) { println("Next: " + it.value); }
    else if(rx.Notification.Kind.OnCompleted == it.kind) { println("Completed"); }
    else if(rx.Notification.Kind.OnError == it.kind) { println("Error: " + it.exception); } },
  { println("Error: " + it.getMessage()); }, // onError
  { println("Sequence complete"); }          // onCompleted
);
```
```
Next: 1
Next: 2
Next: 3
Completed
Sequence complete
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

numbers.materialize().dematerialize().subscribe(
  { println(it); },                          // onNext
  { println("Error: " + it.getMessage()); }, // onError
  { println("Sequence complete"); }          // onCompleted
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
* RxJS: <a href="https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md#rxobservableprototypedematerialize">`dematerialize`</a>
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
  { println(it.toString()); },               // onNext
  { println("Error: " + it.getMessage()); }, // onError
  { println("Sequence complete"); }          // onCompleted
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
* RxJS: <a href="https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md#rxobservableprototypetimestampscheduler">`timestamp`</a>
* <a href="http://www.introtorx.com/Content/v1.0.10621.0/08_Transformation.html#TimeStampAndTimeInterval">Introduction to Rx: TimeStamp and TimeInterval</a>

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
  { println(it.toString()); },              // onNext
  { println("Error:" + it.getMessage()); }, // onError
  { println("Sequence complete"); }         // onCompleted
);
myObservable.subscribe(
  { println(it.toString()); },              // onNext
  { println("Error:" + it.getMessage()); }, // onError
  { println("Sequence complete"); }         // onCompleted
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
  { println(it.toString()); },              // onNext
  { println("Error:" + it.getMessage()); }, // onError
  { println("Sequence complete"); }         // onCompleted
);
myObservable.subscribe(
  { println(it.toString()); },              // onNext
  { println("Error:" + it.getMessage()); }, // onError
  { println("Sequence complete"); }         // onCompleted
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

The `cache( )` method will not itself trigger the execution of the source Observable; an initial observer must subscribe to the Observable returned from `cache( )` before it will begin emitting items.

#### see also:
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#cache()">`cache()`</a>

***

## observeOn( )
#### specify on which Scheduler an Observer should observe the Observable
[[images/rx-operators/observeOn.png]]
To specify in which Scheduler (thread) the Observable should invoke the Observers' `onNext( )`, `onCompleted( )`, and `onError( )` methods, call the Observable's `observeOn( )` method, passing it the appropriate `Scheduler`.

#### see also:
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#observeOn(rx.Scheduler)">`observeOn(scheduler)`</a>
* RxJS: <a href="https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md#rxobservableprototypeobserveonscheduler">`observeOn`</a>
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
* RxJS: <a href="https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md#rxobservableprototypesubscribeonscheduler">`subscribeOn`</a>
* Linq: <a href="http://msdn.microsoft.com/en-us/library/system.reactive.linq.observable.subscribeon.aspx">`SubscribeOn`</a>
* <a href="http://channel9.msdn.com/Series/Rx-Workshop/Rx-Workshop-Schedulers">Rx Workshop: Schedulers</a>
* <a href="http://www.introtorx.com/Content/v1.0.10621.0/15_SchedulingAndThreading.html#SubscribeOnObserveOn">Introduction to Rx: SubscribeOn and ObserveOn</a>

***

## parallel( )
#### split the work done on the emissions from an Observable into multiple Observables each operating on its own parallel thread
[[images/rx-operators/parallel.png]]

You can use the `parallel( )` method to split an Observable into as many Observables as there are available processors, and to do work in parallel on each of these Observables. `parallel( )` will then merge the results of these parallel computations back into a single, well-behaved Observable sequence.

#### see also:
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#parallel(rx.util.functions.Func1)">`parallel(function)`</a>
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#parallel(rx.util.functions.Func1, rx.Scheduler)">`parallel(function,scheduler)`</a>

***

## doOnEach( )
#### register an action to take whenever an Observable emits an item
[[images/rx-operators/doOnEach.png]]

Use the `doOnEach( )` method to register an `Action` that RxJava will perform each time the Observable emits an item. This action takes the item as a parameter.

#### see also:
* Linq: <a href="http://msdn.microsoft.com/en-us/library/system.reactive.linq.observable.do.aspx">`Do`</a>
* RxJS: <a href="https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md#rxobservableprototypedoobserver--onnext-onerror-oncompleted">`do` and `doAction`</a>

***

## doOnCompleted( )
#### register an action to take when an Observable completes successfully
[[images/rx-operators/doOnCompleted.png]]

Use the `doOnCompleted( )` method to register an `Action` that RxJava will perform if the Observable completes normally (not by means of an error).

#### see also:
* Linq: <a href="http://msdn.microsoft.com/en-us/library/system.reactive.linq.observable.do.aspx">`Do`</a>
* RxJS: <a href="https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md#rxobservableprototypedoobserver--onnext-onerror-oncompleted">`do` and `doAction`</a>

***

## doOnError( )
#### register an action to take when an Observable completes with an error
[[images/rx-operators/doOnError.png]]

Use the `doOnError( )` method to register an `Action` that RxJava will perform if the Observable terminates with an error. This action takes the Throwable representing the error as a parameter.

#### see also:
* Linq: <a href="http://msdn.microsoft.com/en-us/library/system.reactive.linq.observable.do.aspx">`Do`</a>
* RxJS: <a href="https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md#rxobservableprototypedoobserver--onnext-onerror-oncompleted">`do` and `doAction`</a>

***

## finallyDo( )
#### register an action to take when an Observable completes
[[images/rx-operators/finallyDo.png]]

You can use the `finallyDo( )` method of an Observable to register an action that RxJava will invoke after that Observable invokes either the `onCompleted( )` or `onError( )` method of its Observer.

```groovy
def numbers = Observable.from([1, 2, 3, 4, 5]);

numbers.finallyDo({ println('Finally'); }).subscribe(
   { println(it); },                          // onNext
   { println("Error: " + it.getMessage()); }, // onError
   { println("Sequence complete"); }          // onCompleted
);
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
* RxJS: <a href="https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md#rxobservableprototypefinallyaction">`finally` / `finallyAction`</a>
* Linq: <a href="http://msdn.microsoft.com/en-us/library/hh212133.aspx">`Finally`</a>

***

## delay( )
#### shift the emissions from an Observable forward in time by a specified amount
[[images/rx-operators/delay.png]]

The `delay( )` operator modifies its source Observable by pausing for a particular increment of time (that you specify) before emitting each of the source Observable's items. This has the effect of shifting the entire sequence of items emitted by the Observable forward in time by that specified increment.

Note that `delay( )` will _not_ time-shift an `onError( )` call in this fashion but it will forward such a call immediately to its subscribers.

#### see also:
* RxJS: <a href="https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md#rxobservableprototypedelayduetime-scheduler">`delay`</a>
* Linq: <a href="http://msdn.microsoft.com/en-us/library/system.reactive.linq.observable.delay.aspx">`Delay`</a>
* <a href="http://www.introtorx.com/Content/v1.0.10621.0/13_TimeShiftedSequences.html#Delay">Introduction to Rx: Delay</a>

***

## delaySubscription( )
#### hold an Observer's subscription request for a specified amount of time before passing it on to the source Observable
[[images/rx-operators/delaySubscription.png]]

The `delaySubscription( )` operator shifts waits for a specified period of time after receiving a subscription request before subscribing to the source Observable.

***

## timeInterval( )
#### emit the time lapsed between consecutive emissions of a source Observable
[[images/rx-operators/timeInterval.png]]

The `timeInterval( )` operator converts a source Observable into an Observable that emits the amount of time lapsed between consecutive emissions of the source Observable. The first emission is the amount of time lapsed between the time the Observer subscribed to the Observable and the time the source Observable emitted its first item. There is no corresponding emission marking the amount of time lapsed between the last emission of the source Observable and the subsequent call to `onCompleted( )`.

#### see also:
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#timeInterval()">`timeInterval()`</a>
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#timeInterval(rx.Scheduler)">`timeInterval(scheduler)`</a>
* RxJS: <a href="https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md#rxobservableprototypetimeintervalscheduler">`timeInterval`</a>
* Linq: <a href="http://msdn.microsoft.com/en-us/library/system.reactive.linq.observable.timeinterval.aspx">`TimeInterval`</a>
* <a href="http://www.introtorx.com/Content/v1.0.10621.0/08_Transformation.html#TimeStampAndTimeInterval">Introduction to Rx: TimeStamp and TimeInterval</a>

***

## using( )
#### create a disposable resource that has the same lifespan as an Observable
[[images/rx-operators/using.png]]

Pass the `using( )` method two factory functions: the first creates a disposable resource, the second creates an Observable. When an observer subscribes to the resulting Observable, `using( )` will use the Observable factory function to create the Observable the observer will observe, while at the same time using the resource factory function to create a resource. When the Observer unsubscribes from the Observable, or when the Observable terminates (normally or with an error), `using( )` will dispose of the resource it created.

#### see also:
* RxJS: <a href="https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md#rxobservableusingresourcefactory-observablefactory">`using`</a>