This section explains operators you can use to combine multiple Observables.

* [**`startWith( )`**](Combining-Observables#startwith) — emit a specified sequence of items before beginning to emit the items from the Observable
* [**`concat( )`**](Combining-Observables#concat) — concatenate two or more Observables sequentially
* [**`merge( )`**](Combining-Observables#merge) — combine multiple Observables into one
* [**`mergeDelayError( )`**](Combining-Observables#mergedelayerror) — combine multiple Observables into one, allowing error-free Observables to continue before propagating errors
* [**`parallelMerge( )`**](Combining-Observables#parallelmerge) — combine multiple Observables into a smaller number of Observables, to facilitate parallelism
* [**`zip( )`**](Combining-Observables#zip) — combine sets of items emitted by two or more Observables together via a specified function and emit items based on the results of this function
* [**`and( )`, `then( )`, and `when( )`**](Combining-Observables#and-then-and-when) — combine sets of items emitted by two or more Observables by means of `Pattern` and `Plan` intermediaries
* [**`combineLatest( )`**](Combining-Observables#combinelatest) — when an item is emitted by either of two Observables, combine the latest item emitted by each Observable via a specified function and emit items based on the results of this function
* [**`join( )`**](Combining-Observables#join) — combine the items emitted by two Observables whenever one item from one Observable falls within a window of duration specified by an item emitted by the other Observable
* [**`switchOnNext( )`**](Combining-Observables#switchonnext) — convert an Observable that emits Observables into a single Observable that emits the items emitted by the most-recently emitted of those Observables
* [**`takeUntil( )`**](Combining-Observables#takeuntil) — emits the items from the source Observable until a second Observable emits an item
* [**`amb( )`**](Combining-Observables#amb) — given two or more source Observables, emits all of the items from the first of these Observables to emit an item

***

## startWith( )
#### emit a specified sequence of items before beginning to emit the items from the Observable

[[images/rx-operators/startWith.png]]

If you want an Observable to immediately begin emitting a specific sequence of items before it begins emitting the items normally expected from it, pass that specific sequence of items into that Observable's `startWith( )` method, as in the following example:
```groovy
def myObservable = Observable.from([1, 2, 3]);

myObservable.startWith(-3, -2, -1, 0).subscribe(
  { println(it); },                  // onNext
  { println("Error encountered"); }, // onError
  { println("Sequence complete"); }  // onCompleted
);
```
```
-3
-2
-1
0
1
2
3
```

#### see also:
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#startWith(T...)">`startWith(x, y, ...)`</a>
* RxJS: <a href="https://github.com/Reactive-Extensions/RxJS/blob/master/doc/observable.md#rxobservableprototypestartwithscheduler-args">`startWith`</a>
* Linq: <a href="http://msdn.microsoft.com/en-us/library/system.reactive.linq.observable.startwith.aspx">`StartWith`</a>
* <a href="http://www.introtorx.com/Content/v1.0.10621.0/12_CombiningSequences.html#StartWith">Introduction to Rx: StartWith</a>

***

## concat( )
#### concatenate two or more Observables sequentially

[[images/rx-operators/concat.png]]

You can concatenate the output of multiple Observables so that they act like a single Observable, with all of the items emitted by the first Observable being emitted before any of the items emitted by the second Observable, by using the `concat( )` method:

```groovy
myConcatenatedObservable = Observable.concat(observable1, observable2, ... );
```

For example, the following code concatenates the 'odds' and 'evens' Observables into a single Observable:

```groovy
odds  = Observable.from([1, 3, 5, 7]);
evens = Observable.from([2, 4, 6]);

Observable.concat(odds, evens).subscribe(
  { println(it); },                  // onNext
  { println("Error encountered"); }, // onError
  { println("Sequence complete"); }  // onCompleted
)
```
```
1
3
5
7
2
4
6
Sequence complete
```

Instead of passing multiple Observables into `concat( )`, you could also pass in a `List<>` of Observables, or even an Observable that emits Observables, and `concat( )` will concatenate their output into the output of a single Observable.

#### see also:
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#concat(rx.Observable...)">`concat(observable1, observable2, ...)`</a>
* RxJS: <a href="https://github.com/Reactive-Extensions/RxJS/blob/master/doc/observable.md#rxobservableprototypeconcatargs">`concat`</a>
* Linq: <a href="http://msdn.microsoft.com/en-us/library/system.reactive.linq.observable.concat.aspx">`Concat`</a>
* <a href="http://www.introtorx.com/Content/v1.0.10621.0/12_CombiningSequences.html#Concat">Introduction to Rx: Concat</a>

***

## merge( )
#### combine multiple Observables into one

[[images/rx-operators/merge.png]]

You can combine the output of multiple Observables so that they act like a single Observable, by using the `merge( )` method:

```groovy
myMergedObservable = Observable.merge(observable1, observable2, ... )
```

For example, the following code merges the `odds` and `evens` Observables into a single Observable:

```groovy
odds  = Observable.from([1, 3, 5, 7]);
evens = Observable.from([2, 4, 6]);

Observable.merge(odds,evens).subscribe(
  { println(it); },                  // onNext
  { println("Error encountered"); }, // onError
  { println("Sequence complete"); }  // onCompleted
);
```
```
1
3
2
5
4
7
6
Sequence complete
```

The items emitted by the merged Observable may appear in any order, regardless of which source Observable they came from.

Instead of passing multiple Observables into `merge( )`, you could also pass in a `List<>` of Observables, or even an Observable that emits Observables, and `merge( )` will merge their output into the output of a single Observable.

If any of the individual Observables passed into `merge( )` aborts by invoking `onError`, the `merge( )` call itself will immediately abort and invoke `onError`. If you would prefer a merge that continues emitting the results of the remaining, error-free Observables before reporting the error, use `mergeDelayError( )` instead.

#### see also:
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#merge(java.util.List)">`merge(listOfObservables)`</a>
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#merge(rx.Observable)">`merge(observableThatEmitsObservables)`</a>
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#merge(rx.Observable...)">`merge(observable1, observable2, ...)`</a>
* RxJS: <a href="https://github.com/Reactive-Extensions/RxJS/blob/master/doc/observable.md#rxobservableprototypemergemaxconcurrent--other">`merge`</a>
* RxJS: <a href="https://github.com/Reactive-Extensions/RxJS/blob/master/doc/observable.md#rxobservableprototypemergeobservable">`mergeObservable`</a>
* Linq: <a href="http://msdn.microsoft.com/en-us/library/system.reactive.linq.observable.merge.aspx">`Merge`</a>
* <a href="http://www.introtorx.com/Content/v1.0.10621.0/12_CombiningSequences.html#Merge">Introduction to Rx: Merge</a>

***

## mergeDelayError( )
#### combine multiple Observables into one but delay errors until completion

[[images/rx-operators/mergeDelayError.png]]

`mergeDelayError( )` behaves much like `merge( )`. The exception is when one of the Observables being merged throws an error. If this happens with `merge( )`, the merged Observable will immediately throw an error itself (that is, it will invoke the `onError` method of its Observer). `mergeDelayError( )`, on the other hand, will hold off on reporting the error until it has given any other non-error-producing Observables that it is merging a chance to finish emitting their items, and it will emit those itself, and will only invoke `onError` when all of the other merged Observables have finished.

Because it is possible that more than one of the merged observables encountered an error, `mergeDelayError( )` may pass information about multiple errors to the `onError` method (which it will never invoke more than once). For this reason, if you want to know the nature of these errors, you should write your `onError` method so that it accepts a parameter of the class `CompositeException`.

#### see also:
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#mergeDelayError(java.util.List)">`mergeDelayError(listOfObservables)`</a>
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#mergeDelayError(rx.Observable)">`mergeDelayError(observableThatEmitsObservables)`</a>
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#mergeDelayError(rx.Observable...)">`mergeDelayError(observable1, observable2, ...)`</a>

***

## parallelMerge( )
#### combine multiple Observables into a smaller number of Observables, to facilitate parallelism
[[images/rx-operators/parallelMerge.png]]

Use the `parallelMerge( )` method to take an Observable that emits a large number of Observables and to reduce it to an Observable that emits a particular, smaller number of Observables that emit the same set of items as the original larger set of Observables: for instance a number of Observables that matches the number of parallel processes that you want to use when processing the emissions from the complete set of Observables.

***

## zip( )
#### combine Observables together via a specified function and emit items based on the results of this function
[[images/rx-operators/zip.png]]

The `zip( )` method returns an Observable that applies a function of your choosing to the combination of items emitted, in sequence, by two (or more) other Observables, with the results of this function becoming the items emitted by the returned Observable. It applies this function in strict sequence, so the first item emitted by the new zip-Observable will be the result of the function applied to the first item emitted by Observable #1 and the first item emitted by Observable #2; the second item emitted by the new zip-Observable will be the result of the function applied to the second item emitted by Observable #1 and the second item emitted by Observable #2; and so forth.

```groovy
myZipObservable = Observable.zip(observable1, observable2, { response1, response2 -> some operation on those responses } );
```

There are also versions of `zip( )` that accept three or four Observables:

```groovy
myZip3Observable = Observable.zip(observable1, observable2, observable3 { response1, response2, response3 -> some operation on those responses });
myZip4Observable = Observable.zip(observable1, observable2, observable3, observable4 { response1, response2, response3, response4 -> some operation on those responses });
```

For example, the following code zips together two Observables, one of which emits a series of odd integers and the other of which emits a series of even integers:

```groovy
odds  = Observable.from([1, 3, 5, 7, 9]);
evens = Observable.from([2, 4, 6]);

Observable.zip(odds, evens, {o, e -> [o, e]}).subscribe(
  { odd, even -> println("odd: " + odd + ", even: " + even); }, // onNext
  { println("Error encountered"); },                            // onError
  { println("Sequence complete"); }                             // onCompleted
)
```
```
odd: 1, even: 2
odd: 3, even: 4
odd: 5, even: 6
Sequence complete
```

**Note:** that the zipped Observable completes normally after emitting three items, which is the number of items emitted by the smaller of the two component Observables (`evens`, which emits three even integers).

#### see also:
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#zip(java.util.Collection, rx.util.functions.FuncN)">`zip()`</a> (several versions)
* RxJS: <a href="https://github.com/Reactive-Extensions/RxJS/blob/master/doc/observable.md#rxobservableprototypezipargs-resultselector">`zip`</a>
* Linq: <a href="http://msdn.microsoft.com/en-us/library/system.reactive.linq.observable.zip.aspx">`Zip`</a>
* <a href="http://www.introtorx.com/Content/v1.0.10621.0/12_CombiningSequences.html#Zip">Introduction to Rx: Zip</a>

***
# and( ), then( ), and when( )
#### combine sets of items emitted by two or more Observables by means of `Pattern` and `Plan` intermediaries
[[images/rx-operators/and_then_when.png]]

The combination of `and( )`, `then( )`, and `when( )` methods operate much like `zip( )` but they do so by means of intermediary data structures.  `and( )` accepts two or more Observables and combines the emissions from each, one set at a time, into `Pattern` objects. `then( )` operates on such `Pattern` objects, transforming them in a `Plan`. `when( )` then transforms these various `Plan` objects into emissions from an Observable.

#### see also:
* Intro to Rx: <a href="http://www.introtorx.com/content/v1.0.10621.0/12_CombiningSequences.html#AndThenWhen">And-Then-When</a>
* Linq: <a href="http://msdn.microsoft.com/en-us/library/hh229153.aspx">And</a>
* Linq: <a href="http://msdn.microsoft.com/en-us/library/hh211662.aspx">Then</a>
* Linq: <a href="http://msdn.microsoft.com/en-us/library/hh229889.aspx">When</a>

***

## combineLatest( )
#### when an item is emitted by either of two Observables, combine the latest item emitted by each Observable via a specified function and emit items based on the results of this function
[[images/rx-operators/combineLatest.png]]

`combineLatest( )` behaves in a similar way to `zip( )`, but while `zip( )` emits items only when all of the zipped source Observables have emitted a previously unzipped item, `combineLatest( )` emits an item whenever _any_ of the source Observables emits an item (so long as each of the source Observables has emitted at least one item). When any of the source Observables emits an item, `combineLatest( )` combines the most recently emitted items from each of the other source Observables, using the function you provide, and emits the return value from that function.

#### see also:
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#combineLatest(rx.Observable, rx.Observable, rx.util.functions.Func2)">`combineLatest(observable1, observable2, combineFunction)`</a> (along with versions that take anywhere from three to nine observables)
* RxJS: <a href="https://github.com/Reactive-Extensions/RxJS/blob/master/doc/observable.md#rxobservableprototypecombinelatestargs-resultselector">`combineLatest`</a>
* Linq: <a href="http://msdn.microsoft.com/en-us/library/hh211991.aspx">`CombineLatest`</a>
* <a href="http://www.introtorx.com/Content/v1.0.10621.0/12_CombiningSequences.html#CombineLatest">Introduction to Rx: CombineLatest</a>

***

## join( )
#### combine the items emitted by two Observables whenever one item from one Observable falls within a window of duration specified by an item emitted by the other Observable
[[images/rx-operators/join_.png]]

The `join( )` method combines the items emitted by two Observables, and selects which items to combine based on duration-windows that you define on a per-item basis. You implement these windows as Observables whose lifespans begin with each item emitted by either Observable. When such a window-defining Observable either emits an item or completes, the window for the item it is associated with closes. So long as an item's window is open, it will combine with any item emitted by the other Observable. You define the function by which the items combine.

#### see also:
* <a href="http://www.introtorx.com/Content/v1.0.10621.0/17_SequencesOfCoincidence.html#Join">Introduction to Rx: Join</a>
* Linq: <a href="http://msdn.microsoft.com/en-us/library/hh229750.aspx">`Join`</a>

***

## switchOnNext( )
#### convert an Observable that emits Observables into a single Observable that emits the items emitted by the most-recently emitted of those Observables
[[images/rx-operators/switchDo.png]]

`switchOnNext( )` subscribes to an Observable that emits Observables. Each time it observes one of these emitted Observables, the Observable returned by `switchOnNext( )` begins emitting items from that Observable. When a new Observable is emitted, `switchOnNext( )` stops emitting items from the earlier-emitted Observable and begins emitting items from the new one.

#### see also:
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#switchOnNext(rx.Observable)">`switchOnNext(sequenceOfSequences)`</a>
* RxJS: <a href="https://github.com/Reactive-Extensions/RxJS/blob/master/doc/observable.md#rxobservableprototypeswitch">`switchLatest`</a>
* Linq: <a href="http://msdn.microsoft.com/en-us/library/hh229197.aspx">`Switch`</a>
* <a href="http://www.introtorx.com/Content/v1.0.10621.0/12_CombiningSequences.html#Switch">Introduction to Rx: Switch</a>

***

## takeUntil( )
#### emits the items from the source Observable until another Observable emits an item
[[images/rx-operators/takeUntil.png]]

#### see also:
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#takeUntil(rx.Observable)">`takeUntil(other)`</a>
* RxJS: <a href="https://github.com/Reactive-Extensions/RxJS/blob/master/doc/observable.md#rxobservableprototypetakeuntilother">`takeUntil`</a>
* Linq: <a href="http://msdn.microsoft.com/en-us/library/hh229530.aspx">`TakeUntil`</a>
* <a href="http://www.introtorx.com/Content/v1.0.10621.0/12_CombiningSequences.html#Amb">Introduction to Rx: Amb</a>
***

## amb( )
#### given two or more source Observables, emits all of the items from the first of these Observables to emit an item
[[images/rx-operators/amb.png]]

When you pass a number of source Observables to `amb( )`, it will pass through the emissions and messages of exactly one of these Observables: the first one that emits an item to `amb( )`. It will ignore and discard the emissions of all of the other source Observables.

#### see also:
* RxJS: <a href="https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md#rxobservableambargs">`amb`</a>
* Linq: <a href="http://msdn.microsoft.com/en-us/library/system.reactive.linq.observable.amb(v=vs.103).aspx">`Amb`</a>