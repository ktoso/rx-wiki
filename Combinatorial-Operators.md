This section explains operators used for combining sequences together.

## concat()

#### Concatenate two or more Observables sequentially

[[images/operation-concat.png]]

You can concatenate the output of multiple Observables so that they act like a single Observable, with all of the items emitted by the first Observable being emitted before any of the items emitted by the second Observable, by using the `Observable.concat()` method:

```groovy
myConcatenatedObservable = Observable.concat(observable1, observable2, … );
```

For example, the following code concatenates the 'odds' and 'evens' Observables into a single Observable:

```groovy
odds  = Observable.toObservable([1, 3, 5, 7]);
evens = Observable.toObservable([2, 4, 6]);

Observable.concat(odds, evens).subscribe(
  [ onNext:{ response.getWriter().println(it); },
    onCompleted:{ response.getWriter().println("Sequence complete"); },
    onError:{ response.getWriter().println("Error encountered"); } ]
)

1
3
5
7
2
4
6
Sequence complete
```

Instead of passing multiple Observables into `Observable.concat()`, you could also pass in a `List<>` of Observables, or even a Observable that emits Observables, and `Observable.concat()` will concatenate their output into the output of a single Observable.




## `Observable.merge( )`

[[images/operation-merge.png]]

You can combine the output of multiple Observables so that they act like a single Observable, by using the `merge( )` method:

```groovy
`myMergedObservable = Observable.merge( *observable1*, *observable2*, … )`
```

For example, the following code merges the `odds` and `evens` Observables into a single Observable:

```groovy
odds  = Observable.toObservable( [1, 3, 5, 7] );
evens = Observable.toObservable( [2, 4, 6] );

Observable.merge(odds,evens).subscribe(
  [ onNext:{ response.getWriter().println( it ); },
    onCompleted:{ response.getWriter().println( "Sequence complete" ); },
    onError:{ response.getWriter().println( "Error encountered" ); } ]
);

`1`
`3`
`2`
`5`
`4`
`7`
`6`
`Sequence complete`
```

The items emitted by the merged Observable may appear in any order, regardless of which source Observable they came from.

Instead of passing multiple Observables into `Observable.merge( )`, you could also pass in a `List<>` of Observables, or even a Observable that emits Observables, and `Observable.merge( )` will merge their output into the output of a single Observable.

If any of the individual Observables passed into `Observable.merge( )` aborts by calling `onError`, the `Observable.merge( )` call itself will immediately abort and call `onError`. If you would prefer a merge that continues emitting the results of the remaining, error-free Observables before reporting the error, use `Observable.mergeDelayError( )` instead.

## `Observable.mergeDelayError( )`

[[images/operation-mergeDelayError.png]]

`Observable.mergeDelayError( )` works much like `Observable.merge( )`. The exception is when one of the Observables being merged throws an error. If this happens with `Observable.merge( )`, the merged Observable will immediately throw an error itself (that is, it will call the `onError` closure of its observer). `Observable.mergeDelayError( )`, on the other hand, will hold off on reporting the error until it has given any other non-error-producing Observables being merged a chance to finish emitting their items, and will emit those itself, only calling `onError` when all of the other merged Observables have finished.

Because it is possible that more than one of the merged observables encountered an error, `Observable.mergeDelayError( )` may pass information about multiple errors to the `onError` closure, which it will never call more than once. For this reason, if you want to know the nature of these errors, you should write the `onError` closure so that it accepts a parameter of the class `CompositeException`.




## `Observable.zip( )`

[[images/operation-zip.png]]

The `zip( )` method returns a Observable that applies a closure of your choosing to the combination of items emitted, in sequence, by two (or more) other Observables, with the results of this closure becoming the sequence emitted by the returned Observable. It applies this closure in strict sequence, so the first object emitted by the new Observable will be the result of the closure applied to the first object emitted by Observable #1 and the first object emitted by Observable #2; the second object emitted by the new Observable will be the result of the closure applied to the second object emitted by Observable #1 and the second object emitted by Observable #2; and so forth.

```groovy
`myZipObservable = Observable.zip( *observable1*, *observable2*, \{ *response1*, *response2* \-> *some operation on those responses* \} );`
```

There are also versions of `zip( )` that accept three or four Observables:

```groovy
`myZipObservable = Observable.zip( *observable1*, *observable2*, *observable3* \{ *response1*, *response2*, *response3* \-> *some operation on those responses* \} );`
`myZipObservable = Observable.zip( *observable1*, *observable2*, *observable3*, *observable4* \{ *response1*, *response2*, *response3*, *response4* \-> *some operation on those responses* \} );`
```

For example, the following code zips together two Observables, one of which emits a series of odd integers and the other of which emits a series of even integers:

```groovy
odds  = Observable.toObservable( [1, 3, 5, 7, 9] );
evens = Observable.toObservable( [2, 4, 6] );

Observable.zip( odds, evens, {o, e -> [o, e]} ).subscribe(
  [ onNext:{ odd, even -> response.getWriter().println( "odd: " + odd + ", even: " + even); },
    onCompleted:{ response.getWriter().println( "Sequence complete" ); },
    onError:{ response.getWriter().println( "Error encountered" ); } ]
)

`odd: 1, even: 2`
`odd: 3, even: 4`
`odd: 5, even: 6`
`Sequence complete`
```

Note that the zipped Observable completes normally after emitting three items, which is the number of items emitted by the smaller of the two component Observables (`evens`, which emits three even integers).
