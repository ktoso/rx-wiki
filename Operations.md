
# Observable utility methods

The following methods allow you to change the default behavior of a Observable:
| `Observable.onErrorResumeNext( )`|http://go/apidoc/com/netflix/api/platform/reactive/operations/ObservableExtensions.html#onErrorResumeNext(com.netflix.api.platform.reactive.Observable, com.netflix.api.platform.functions.Func1)]^ | instructs a Observable to attempt to continue emitting values after it encounters an error |
| `Observable.onErrorReturn( )`|http://go/apidoc/com/netflix/api/platform/reactive/operations/ObservableExtensions.html#onErrorReturn(com.netflix.api.platform.reactive.Observable, com.netflix.api.platform.functions.Func1)]^ | instructs a Observable to emit a particular value to a observer’s `onNext` closure when it encounters an error |
| `Observable.synchronize( )`|http://go/apidoc/com/netflix/api/platform/reactive/operations/ObservableExtensions.html#synchronize(com.netflix.api.platform.reactive.Observable)]^ | forces a Observable to call its observer’s closures in a "well-behaved" chronological order |

Each of these methods takes a Observable as a parameter, and returns the same Observable with its default behavior altered.

# Methods for transforming Observables

There are several methods that you can use to transform Observables. These methods themselves return Observable objects, and because of this you can chain these methods together in many ways.

These methods are part of the `Observable` class as both instance and static methods.

These utility methods are as follows:

- `Observable.concat( )` – concatenates two or more Observables by emitting all of the items emitted by the first, then all the items emitted by the next, as a single Observable \\  \\  {color:red}TBD, see{color} {color:red}[API-5009
- `Observable.filter( )` – applies a closure that acts as a boolean filter to each object emitted by a Observable, and emits only those objects that pass the filter as a Observable 
- `Observable.join( )`	
- `Observable.last( )` – takes the last item emitted by a Observable and emits it as the sole emission from a Observable 
- `Observable.map( )` – applies a closure to each object emitted by a Observable, and emits the result as a Observable 
- `Observable.mapMany( )`	
- `Observable.mapManyDelayError( )`	
- `*w*.mapManyDelayError( )` – concatenates two or more Observables by emitting all of the items emitted by the first, then all the items emitted by the next, as a single Observable
- `Observable.materialize( )` – converts each `on*Foo{pl`} call from a Observable into an explicit `onNext` emission of the resulting Observable 
- `Observable.merge( )` – combines the objects emitted by two or more Observables, and emits the result as a single Observable 
- `Observable.mergeDelayError( )` – combines the objects emitted by two or more Observables, and emits the result as a single Observable, reserving any errors until all non-error-producing streams have given their complete output 
- `Observable.partition( )` – splits a Observable into two Observables based on the results of a boolean evaluation applied to all the items emitted by the original Observable
- `Observable.reduce( )` – applies a closure to the first item emitted by a Observable, then passes that result into the same closure along with the second item emitted by the Observable, continuing this process and emitting the results of the final such closure call as the sole result of a new Observable 
- `Observable.scan( )` – applies a closure to the first item emitted by a Observable, then passes that result into the same closure along with the second item emitted by the Observable, continuing this process and emitting the results of each closure call as the results of a new Observable 
- `Observable.skip( *n* )` – throws away the first *n* items emitted by a Observable and emits only those that come after 
- `Observable.take( *n* )` – takes the first *n* items emitted by a Observable and emits these as a Observable 
- `Observable.toList( )` – converts the set of objects emitted by a Observable into a single list object 
- `Observable.toSortedList( )` – converts the set of objects emitted by a Observable into a single list object, sorted numerically or by a function 
- `Observable.zip( )` – applies a closure to each object emitted by two or more Observables, taken together, in sequence, and emits the result as a Observable 

The following sections will describe these methods in greater detail.

## `Observable.concat( )`

[[images/operation-concat.png]]

You can concatenate the output of multiple Observables so that they act like a single Observable, with all of the items emitted by the first Observable being emitted before any of the items emitted by the second Observable, by using the `Observable.concat( )` method:

```groovy
`myConcatenatedObservable = Observable.concat( *observable1*, *observable2*, … );`
```

For example, the following code concatenates the `odds` and `evens` Observables into a single Observable:

```groovy
odds  = Observable.toObservable( [1, 3, 5, 7] );
evens = Observable.toObservable( [2, 4, 6] );

Observable.concat( odds, evens ).subscribe(
  [ onNext:{ response.getWriter().println( it ); },
    onCompleted:{ response.getWriter().println( "Sequence complete" ); },
    onError:{ response.getWriter().println( "Error encountered" ); } ]
)

`1`
`3`
`5`
`7`
`2`
`4`
`6`
`Sequence complete`
```

Instead of passing multiple Observables into `Observable.concat( )`, you could also pass in a `List<>` of Observables, or even a Observable that emits Observables, and `Observable.concat( )` will concatenate their output into the output of a single Observable.

## `Observable.filter( )` {anchor:filter}

[[images/operation-filter.png]]

You can filter a Observable, discarding any values that do not meet some test, by passing a filtering closure into the `Observable.filter( )` method. For example, the following code filters a list of integers, emitting only those that are even (that is, where the remainder from dividing the number by two is zero):

```groovy
numbers = Observable.toObservable( [1, 2, 3, 4, 5, 6, 7, 8, 9] );

Observable.filter( numbers, { 0 == (it % 2) } ).subscribe(
  [ onNext:{ response.getWriter().println( it ); },
    onCompleted:{ response.getWriter().println( "Sequence complete" ); },
    onError:{ response.getWriter().println( "Error encountered" ); } ]
);

`2`
`4`
`6`
`8`
`Sequence complete`
```

In addition to calling `filter( )` as a stand-alone method, you can also call it as a method of a Observable object, so, in the example above, instead of 

```groovy
Observable.filter( numbers, { 0 == (it %2) } )…
```
you could instead write 
```groovy
numbers.filter( { 0 == (it % 2) } )…
```

## `Observable.last( )`

[[images/operation-last.png]]

To convert a Observable that emits several objects into one that only emits the last of these objects before completing, use the `last( )` method. For instance, in the following code, `last( )` emits only the last integer in the list of integers represented by `numbers`:

```groovy
numbers = Observable.toObservable( [1, 2, 3, 4, 5, 6, 7, 8, 9] );

Observable.last(numbers).subscribe(
  [ onNext:{ response.getWriter().println( it ); },
    onCompleted:{ response.getWriter().println( "Sequence complete" ); },
    onError:{ response.getWriter().println( "Error encountered" ); } ]
);

`9`
`Sequence complete`
```

In addition to calling `last( )` as a stand-alone method, you can also call it as a method of a Observable object, so, in the example above, instead of 

```groovy
Observable.last(numbers)…
``` 
you could instead write

```groovy
numbers.last()…
```

## `Observable.map( )`

[[images/operation-map.png]]

The `map( )` method applies a closure of your choosing to every object emitted by a Observable, and returns this transformation as a new Observable sequence. For example, the following code maps a closure that squares the incoming value onto the values in `numbers`:

```groovy
numbers = Observable.toObservable( [1, 2, 3, 4, 5] );

Observable.map( numbers, { it * it } ).subscribe(
  [ onNext:{ response.getWriter().println( it ); },
    onCompleted:{ response.getWriter().println( "Sequence complete" ); },
    onError:{ response.getWriter().println( "Error encountered" ); } ]
);

`1`
`4`
`9`
`16`
`25`
`Sequence complete`
```

In addition to calling `map( )` as a stand-alone method, you can also call it as a method of a Observable object, so, in the example above, instead of 

```groovy
Observable.map( numbers, { it * it } )…
```
you could instead write 
```groovy
numbers.map( { it * it } )…
```

## `Observable.mapMany( )` and `Observable.mapManyDelayError( )`

[[images/operation-mapMany.png]]

The `mapMany( )` method creates a new Observable sequence by applying a closure that you supply to each object in the original Observable sequence, where that closure is itself a Observable that emits objects, and then merges the results of that closure applied to every item emitted by the original Observable, emitting these merged results as its own sequence.

It is useful, for example, when you have a Observable that emits a series of objects that themselves have Observable members or are in other ways transformable into Observables, so that you can create a new Observable that emits the complete collection of items emitted by the sub-Observables of these objects.

```groovy
numbers   = Observable.toObservable( [1, 2, 3] );                    // this closure is a Observable that emits three numbers
multiples = { n -> Observable.toObservable( [ n*1, n*2, n*3 ] ) };   // this closure is a Observable that emits three numbers based on what number it is passed

numbers.mapMany( multiples ).subscribe(
  [ onNext:{ response.getWriter().println( it.toString() ); },
    onCompleted:{ response.getWriter().println( "Sequence complete" ); },
    onError:{ response.getWriter().println( "Error encountered" ); } ]
);

`1`
`2`
`3`
`2`
`4`
`6`
`3`
`6`
`9`
`Sequence complete`
```

If any of the individual Observables mapped to the emissions from the source Observable in `Observable.mapMany( )` aborts by calling `onError`, the `Observable.mapMany( )` call itself will immediately abort and call `onError`. If you would prefer that the map-many continue emitting the results of the remaining, error-free Observables before reporting the error, use `Observable.mapManyDelayError( )` instead.

Because it is possible that more than one of the individual observables encountered an error, `Observable.mapManyDelayError( )` may pass information about multiple errors to the `onError` closure, which it will never call more than once. For this reason, if you want to know the nature of these errors, you should write the `onError` closure so that it accepts a parameter of the class `CompositeException`.

## `Observable.materialize( )`

[[images/operation-materialize.png]]

A well-formed Observable will call its observer’s `onNext` closure zero or more times, and then will call either the `onCompleted` or `onError` closure exactly once. The `Observable.materialize( )` method converts this series of calls into a series of emissions from a Observable, where it represents each such call as a `ObservableNotification` object.

For example:

```groovy
numbers = Observable.toObservable( [1, 2, 3] );

Observable.materialize( numbers ).subscribe(
  [ onNext: { if( Kind.OnNext == it.kind ) response.getWriter().println( "Next: " + it.value );
              else if( Kind.OnCompleted == it.kind ) response.getWriter().println( "Completed" );
              else if( Kind.OnError == it.kind ) response.getWriter().println( "Error: " + it.exception ); },
    onCompleted:{ response.getWriter().println( "Sequence complete" ); },
    onError:{ response.getWriter().println( "Error encountered" ); } ]
);

`Next: 1`
`Next: 2`
`Next: 3`
`Completed`
`Sequence complete`
```

In addition to calling `materialize( )` as a stand-alone method, you can also call it as a method of a Observable object, so that instead of `Observable.materialize( numbers )…` in the above example, you could also write `numbers.materialize( )…`.

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


## `Observable.reduce( )`

[[images/operation-reduce.png]]

The `reduce( )` method returns a Observable that applies a closure of your choosing to the first item emitted by a source Observable, then feeds the result of that closure along with the second item emitted by the source Observable into the same closure, then feeds the result of _that_ closure along with the third item into the same closure, and so on until all items have been emitted by the source Observable. Then it emits the final result from the final call to your closure as the sole output from the returned Observable.

This technique, which is called "reduce" here, is sometimes called "fold," "accumulate," "compress," or "inject" in other programming contexts. Groovy itself has an `inject( )` method that does a similar operation on lists.

For example, the following code uses `reduce( )` to compute, and then emit as a Observable, the sum of the numbers emitted by the source Observable:

```groovy
numbers = Observable.toObservable( [1, 2, 3, 4, 5] );

Observable.reduce( numbers, { a, b -> a+b } ).subscribe(
  [ onNext:{ response.getWriter().println( it ); },
    onCompleted:{ response.getWriter().println( "Sequence complete" ); },
    onError:{ response.getWriter().println( "Error encountered" ); } ]
);

`15`
`Sequence complete`
```

In addition to calling `reduce( )` as a stand-alone method, you can also call it as a method of a Observable object, so, in the example above, instead of 

```groovy
Observable.reduce( numbers, { a, b -> a+b } )…
```
you could instead write 

```groovy
numbers.reduce( { a, b -> a+b } )…
```

There is also a version of `reduce( )` to which you can pass a seed value in addition to an accumulator function:

```groovy
`Observable.reduce( *observable*, *initial_seed*, *accumulator_closure* )`
or
`*observable*.reduce( *initial_seed*, *accumulator_closure* )`
```

## `Observable.scan( )`

[[images/operation-scan.png]]

The `scan( )` method returns a Observable that applies a closure of your choosing to the first item emitted by a source Observable, then feeds the result of that closure along with the second item emitted by the source Observable into the same closure, then feeds the result of that closure along with the third item into the same closure, and so on until all items have been emitted by the source Observable. It emits the result of each of these iterations as a sequence from the returned Observable. This sort of closure is sometimes called an _accumulator_.

For example, the following code takes a Observable that emits a consecutive sequence of *n* integers starting with 1 and converts it into a Observable that emits the first *n* [triangular numbers|http://en.wikipedia.org/wiki/Triangular_number]:

```groovy
numbers = Observable.toObservable( [1, 2, 3, 4, 5] );

Observable.scan(numbers, { a, b -> a+b } ).subscribe(
  [ onNext:{ response.getWriter().println( it ); },
    onCompleted:{ response.getWriter().println( "Sequence complete" ); },
    onError:{ response.getWriter().println( "Error encountered" ); } ]
);

`1`
`3`
`6`
`10`
`15`
`Sequence complete`
```

In addition to calling `scan( )` as a stand-alone method, you can also call it as a method of a Observable object, so, in the example above, instead of 

```groovy
Observable.scan( numbers, { a, b -> a+b } )…
```
you could instead write 
```groovy
numbers.scan( { a, b -> a+b } )…
```

There is also a version of `scan( )` to which you can pass a seed value in addition to an accumulator function:

```groovy
`Observable.scan( *observable*, *initial_seed*, *accumulator_closure* )`
or
`*observable*.scan( *initial_seed*, *accumulator_closure* )`
```

Note that if you pass a seed value to `scan( )`, it will emit the seed itself as its first value.

## `Observable.skip( )`

[[images/operation-skip.png]]

You can ignore the first *n* items emitted by a Observable and attend only to those items that come after, by modifying the Observable with the `Observable.skip( *n* )` method.

```groovy
numbers = Observable.toObservable( [1, 2, 3, 4, 5, 6, 7, 8, 9] );

Observable.skip(numbers, 3).subscribe(
  [ onNext:{ response.getWriter().println( it ); },
    onCompleted:{ response.getWriter().println( "Sequence complete" ); },
    onError:{ response.getWriter().println( "Error encountered" ); } ]
);

`4`
`5`
`6`
`7`
`8`
`9`
`Sequence complete{`}
```

In addition to calling skip( ) as a stand-alone method, you can also call it as a method of a Observable object, so, in the example above, instead of 

```groovy
Observable.skip( numbers, 3 )…
``` you could instead write 

```groovy
numbers.skip(3)…
```

## `Observable.take( )`

[[images/operation-take.png]]

You can choose to pay attention only to the first *n* values emitted by a Observable by calling its `take( *n* )` method. That method returns a Observable that will call a subscribing observer’s `onNext` closure a maximum of *n* times before calling `onCompleted`. For example,

```groovy
numbers = Observable.toObservable( [1, 2, 3, 4, 5, 6, 7, 8, 9] );

Observable.take(numbers, 3).subscribe(
  [ onNext:{ response.getWriter().println( it ); },
    onCompleted:{ response.getWriter().println( "Sequence complete" ); },
    onError:{ response.getWriter().println( "Error encountered" ); } ]
);

`1`
`2`
`3`
`Sequence complete`
```

In addition to calling `take( )` as a stand-alone method, you can also call it as a method of a Observable object, so, in the example above, instead of 

```groovy
Observable.take( numbers, 3 )…
```
you could instead write 
```groovy
numbers.take(3)…
```

If you call `take( *n* )` on a Observable, and that Observable emits _fewer_ than *n* items before completing, the new, `take`\-modified Observable will _not_ throw an error, but will merely emit this same fewer number of items before it completes.

## `Observable.toList( )`

[[images/operation-toList.png]]

Normally, a Observable that emits multiple items will do so by calling its observer’s `onNext` closure for each such item. You can change this behavior, instructing the Observable to compose a list of these multiple items and then to call the observer’s `onNext` closure _once_, passing it the entire list, by calling the Observable object’s `toList( )` method prior to calling its `subscribe( )` method. For example:

```groovy
`Observable.tolist(myObservable).subscribe(\[ onNext: \{ myListOfSomething \-> *do something useful with the list* \} \]);`
```

For example, the following rather pointless code takes a list of integers, converts it into a Observable, then converts that Observable into one that emits the original list as a single item:

```groovy
numbers = Observable.toObservable( [1, 2, 3, 4, 5, 6, 7, 8, 9] );

Observable.toList(numbers).subscribe(
  [ onNext:{ response.getWriter().println( it ); },
    onCompleted:{ response.getWriter().println( "Sequence complete" ); },
    onError:{ response.getWriter().println( "Error encountered" ); } ]
);

`\[1, 2, 3, 4, 5, 6, 7, 8, 9\]`
`Sequence complete`
```

In addition to calling `toList( )` as a stand-alone method, you can also call it as a method of a Observable object, so, in the example above, instead of 
```groovy
Observable.toList( numbers )…
``` 
you could instead write 
```groovy
numbers.toList( )…
```

## `Observable.toSortedList( )`

[[images/operation-toSortedList.png]]

The `toSortedList( )` method behaves much like `toList( )` except that it sorts the resulting list. By default it sorts the list numerically, from lowest to highest, but you can also pass in a function that takes two values and returns a number, and `toSortedList( )` will use that number instead of the numerical difference between the two values to sort the values.

For example, the following code takes a list of unsorted integers, converts it into a Observable, then converts that Observable into one that emits the original list in sorted form as a single item:

```groovy
numbers = Observable.toObservable( [8, 6, 4, 2, 1, 3, 5, 7, 9] );

Observable.toSortedList(numbers).subscribe(
  [ onNext:{ response.getWriter().println( it ); },
    onCompleted:{ response.getWriter().println( "Sequence complete" ); },
    onError:{ response.getWriter().println( "Error encountered" ); } ]
)

`\[1, 2, 3, 4, 5, 6, 7, 8, 9\]`
`Sequence complete`
```

In addition to calling `toList( )` as a stand-alone method, you can also call it as a method of a Observable object, so, in the example above, instead of {code}Observable.toSortedList( numbers )…{code} you could instead write {code}numbers.toSortedList( )…{code}

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
