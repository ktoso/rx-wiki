This section explains operators you can use to combine multiple Observable sequences.

* [**`startWith( )`**](Combining-Observables#startwith) — emit a specified sequence of values before beginning to emit the Observable sequence
* [**`concat( )`**](Combining-Observables#concat) — concatenate two or more Observables sequentially
* [**`merge( )`**](Combining-Observables#merge) — combine multiple Observables into one
* [**`mergeDelayError( )`**](Combining-Observables#mergedelayerror) — combine multiple Observables into one, allowing error-free Observables to continue before propagating errors
* [**`zip( )`**](Combining-Observables#zip) — combine Observables together via a provided closure and emit values based on the results of this closure
* [**`switchDo( )`**](Combining-Observables#switchdo) — convert an Observable sequence of Observables into a single Observable that emits the emissions of the most-recently emitted of the Observables in the sequence
* [**`takeUntil( )`**](Combining-Observables#takeuntil) — emits the values from the source Observable until a second Observable emits a value

## startWith( )
#### emit a specified sequence of values before beginning to emit the Observable sequence

[[images/rx-operators/startWith.png]]

If you want an Observable to immediately begin emitting a specific sequence of values before it begins emitting the values normally expected from it, pass that specific sequence of values into that Observable's `startWith( )` method, as in the following example:
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

If any of the individual Observables passed into `merge( )` aborts by calling `onError`, the `merge( )` call itself will immediately abort and call `onError`. If you would prefer a merge that continues emitting the results of the remaining, error-free Observables before reporting the error, use `mergeDelayError( )` instead.

## mergeDelayError( )
#### combine multiple Observables into one but delay errors until completion

[[images/rx-operators/mergeDelayError.png]]

`mergeDelayError( )` behaves much like `merge( )`. The exception is when one of the Observables being merged throws an error. If this happens with `merge( )`, the merged Observable will immediately throw an error itself (that is, it will call the `onError` closure of its observer). `mergeDelayError( )`, on the other hand, will hold off on reporting the error until it has given any other non-error-producing Observables that it is merging a chance to finish emitting their items, and it will emit those itself, and will only call `onError` when all of the other merged Observables have finished.

Because it is possible that more than one of the merged observables encountered an error, `mergeDelayError( )` may pass information about multiple errors to the `onError` closure (which it will never call more than once). For this reason, if you want to know the nature of these errors, you should write your `onError` closure so that it accepts a parameter of the class `CompositeException`.

## zip( )
#### combine Observables together via a provided closure and emit values based on the results of this closure

[[images/rx-operators/zip.png]]

The `zip( )` method returns an Observable that applies a closure of your choosing to the combination of items emitted, in sequence, by two (or more) other Observables, with the results of this closure becoming the sequence emitted by the returned Observable. It applies this closure in strict sequence, so the first object emitted by the new zip-Observable will be the result of the closure applied to the first object emitted by Observable #1 and the first object emitted by Observable #2; the second object emitted by the new zip-Observable will be the result of the closure applied to the second object emitted by Observable #1 and the second object emitted by Observable #2; and so forth.

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

## switchDo( )
#### convert an Observable sequence of Observables into a single Observable that emits the emissions of the most-recently emitted of the Observables in the sequence

[[images/rx-operators/switchDo.png]]

`switchDo( )` subscribes to an Observable that emits Observables. Each time it observes one of these emitted Observables, the Observable returned by `switchDo( )` begins emitting objects from that Observable. When a new Observable is emitted, `switchDo( )` stops emitting items from the earlier-emitted Observable and begins emitting items from the new one.

## takeUntil( )
#### emits the values from the source Observable until another Observable emits a value

[[images/rx-operators/takeUntil.png]]