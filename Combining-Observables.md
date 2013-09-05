This section explains operators you can use to combine multiple Observables.

* [**`startWith( )`**](Combining-Observables#startwith) — emit a specified sequence of items before beginning to emit the items from the Observable
* [**`concat( )`**](Combining-Observables#concat) — concatenate two or more Observables sequentially
* [**`merge( )`**](Combining-Observables#merge) — combine multiple Observables into one
* [**`mergeDelayError( )`**](Combining-Observables#mergedelayerror) — combine multiple Observables into one, allowing error-free Observables to continue before propagating errors
* [**`zip( )`**](Combining-Observables#zip) — combine sets of items emitted by two or more Observables together via a specified function and emit items based on the results of this function
* [**`combineLatest( )`**](Combining-Observables#combinelatest) — when an item is emitted by either of two Observables, combine the latest item emitted by each Observable via a specified function and emit items based on the results of this function
* [**`switchOnNext( )`**](Combining-Observables#switchonnext) — convert an Observable that emits Observables into a single Observable that emits the items emitted by the most-recently emitted of those Observables
* [**`takeUntil( )`**](Combining-Observables#takeuntil) — emits the items from the source Observable until a second Observable emits an item

## startWith( )
#### emit a specified sequence of items before beginning to emit the items from the Observable

[[images/rx-operators/startWith.png]]

If you want an Observable to immediately begin emitting a specific sequence of items before it begins emitting the items normally expected from it, pass that specific sequence of items into that Observable's `startWith( )` method, as in the following example:
```groovy
def myObservable = Observable.toObservable([1, 2, 3]);

myObservable.startWith(-3, -2, -1, 0).subscribe(
  [ onNext:{ myWriter.println(it); },
    onCompleted:{ myWriter.println("Sequence complete"); },
    onError:{ myWriter.println("Error encountered"); } ]
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

## concat( )
#### concatenate two or more Observables sequentially

[[images/rx-operators/concat.png]]

You can concatenate the output of multiple Observables so that they act like a single Observable, with all of the items emitted by the first Observable being emitted before any of the items emitted by the second Observable, by using the `concat( )` method:

```groovy
myConcatenatedObservable = Observable.concat(observable1, observable2, ... );
```

For example, the following code concatenates the 'odds' and 'evens' Observables into a single Observable:

```groovy
odds  = Observable.toObservable([1, 3, 5, 7]);
evens = Observable.toObservable([2, 4, 6]);

Observable.concat(odds, evens).subscribe(
  [ onNext:{ myWriter.println(it); },
    onCompleted:{ myWriter.println("Sequence complete"); },
    onError:{ myWriter.println("Error encountered"); } ]
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

## merge( )
#### combine multiple Observables into one

[[images/rx-operators/merge.png]]

You can combine the output of multiple Observables so that they act like a single Observable, by using the `merge( )` method:

```groovy
myMergedObservable = Observable.merge(observable1, observable2, ... )
```

For example, the following code merges the `odds` and `evens` Observables into a single Observable:

```groovy
odds  = Observable.toObservable([1, 3, 5, 7]);
evens = Observable.toObservable([2, 4, 6]);

Observable.merge(odds,evens).subscribe(
  [ onNext:{ myWriter.println(it); },
    onCompleted:{ myWriter.println("Sequence complete"); },
    onError:{ myWriter.println("Error encountered"); } ]
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

## mergeDelayError( )
#### combine multiple Observables into one but delay errors until completion

[[images/rx-operators/mergeDelayError.png]]

`mergeDelayError( )` behaves much like `merge( )`. The exception is when one of the Observables being merged throws an error. If this happens with `merge( )`, the merged Observable will immediately throw an error itself (that is, it will invoke the `onError` method of its Observer). `mergeDelayError( )`, on the other hand, will hold off on reporting the error until it has given any other non-error-producing Observables that it is merging a chance to finish emitting their items, and it will emit those itself, and will only invoke `onError` when all of the other merged Observables have finished.

Because it is possible that more than one of the merged observables encountered an error, `mergeDelayError( )` may pass information about multiple errors to the `onError` method (which it will never invoke more than once). For this reason, if you want to know the nature of these errors, you should write your `onError` method so that it accepts a parameter of the class `CompositeException`.

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
odds  = Observable.toObservable([1, 3, 5, 7, 9]);
evens = Observable.toObservable([2, 4, 6]);

Observable.zip(odds, evens, {o, e -> [o, e]}).subscribe(
  [ onNext:{ odd, even -> myWriter.println("odd: " + odd + ", even: " + even); },
    onCompleted:{ myWriter.println("Sequence complete"); },
    onError:{ myWriter.println("Error encountered"); } ]
)
```
```
odd: 1, even: 2
odd: 3, even: 4
odd: 5, even: 6
Sequence complete
```

**Note:** that the zipped Observable completes normally after emitting three items, which is the number of items emitted by the smaller of the two component Observables (`evens`, which emits three even integers).

## combineLatest( )
#### when an item is emitted by either of two Observables, combine the latest item emitted by each Observable via a specified function and emit items based on the results of this function
[[images/rx-operators/combineLatest.png]]

`combineLatest( )` behaves in a similar way to `zip( )`, but while `zip( )` emits items only when all of the zipped source Observables have emitted a previously unzipped item, `combineLatest( )` emits an item whenever _any_ of the source Observables emits an item (so long as each of the source Observables has emitted at least one item). When any of the source Observables emits an item, `combineLatest( )` combines the most recently emitted items from each of the other source Observables, using the function you provide, and emits the return value from that function.

## switchOnNext( )
#### convert an Observable that emits Observables into a single Observable that emits the items emitted by the most-recently emitted of those Observables
[[images/rx-operators/switchDo.png]]

`switchOnNext( )` subscribes to an Observable that emits Observables. Each time it observes one of these emitted Observables, the Observable returned by `switchOnNext( )` begins emitting items from that Observable. When a new Observable is emitted, `switchOnNext( )` stops emitting items from the earlier-emitted Observable and begins emitting items from the new one.

## takeUntil( )
#### emits the items from the source Observable until another Observable emits an item

[[images/rx-operators/takeUntil.png]]