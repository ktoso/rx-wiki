# Introduction

In the Rx (reactive) pattern an _observer_ closure _subscribes_ to an object that implements the _Observable_ interface. Then that Observer closure reacts to whatever object or sequence of objects the Observable object emits. This pattern facilitates concurrent operations by not blocking while waiting for the Observable to emit objects, but by instead creating a sentry in the form of an Observer that stands ready to react appropriately at whatever future time the Observable does so.

This page explains what the reactive pattern is, what Observables and Observers are (and how Observers subscribe to Observables), and how you use utility methods to link Observables together and change their behaviors.


> This guide accompanies its explanations with "marble diagrams." Here is how marble diagrams represent Observables and transformations of Observables:

[[images/operation-legend.png]]


# Background

In many software programming tasks, you more or less expect that the instructions you write will execute and complete incrementally, one-at-a-time, in order as you have written them. So for instance, you might write a program something like the following pseudocode:

```java
catalog       = getCatalog("Montgomery Ward");                           // get a "Montgomery Ward" catalog object
availableCash = myBankAccount.getBalance();                                // get my bank account balance
jeans         = catalog.findJeans("38W", "38L", "blue" );                 // find my size of jeans in the catalog
if( availableCash >= jeans.getPurchasePrice() ) catalog.purchase( jeans ); // if I have enough money, buy them
```

The sequence of events would, predictably and dependably, look like this: 

[[images/jeans-activity1.png]]

But in the Rx paradigm, many instructions execute in parallel and their results are later captured, in arbitrary order, by closures called "observers." In these cases, rather than _calling_ a method, you _define_ a method call in the form of a "Observable," and then _subscribe_ an observer to it, at which point the call takes place in the background with the observer standing sentry to capture and respond to its return values whenever they arrive.

An advantage of this approach is that when you have a bunch of tasks to do that are not dependent on each other, you can start them all at the same time rather than waiting for each one to finish before starting the next one --- that way, your entire bundle of tasks only takes as long to complete as the longest task in the bundle.

An important part of your design process will be to analyze your tasks with an eye to which ones can happen asynchronously and in parallel, and which ones must block because of dependencies on the results of other tasks.

Here is how the equivalent jeans-buying process might take place in the reactive model:
```groovy
catalogObservable = getCatalog("Montgomery Ward");
catalogObservable.mapMany( catalog -> catalog.findJeans("38W", "38L", "blue" ) )
                .zip( myBankAccount.getBalance( ) ) { product, cash -> if( cash > product.getPurchasePrice() ) product.purchase() ) };
```

And rather than a sequence diagram, a marble diagram will be used to illustrate its behavior:

[[images/jeans-marble1.png]]

> There are many terms used to describe this model of asynchronous programming and design. This document will use the following terms: An _observer_ is a closure that you _subscribe_ to an object that implements the _Observable_ interface; that is, you _subscribe_ an _observer_ to an _Observable_.

> In other documents and other contexts, what we are calling a "observer" is sometimes called a "reactor." This model in general is often referred to as the ["reactor pattern"|http://en.wikipedia.org/wiki/Reactor_pattern].


# Setting up Observers

> Groovy will be used for code examples in this document but any JVM language can be used such as Clojure, Scala, JRuby, Javascript or Java itself. 
> For Java itself, anywhere 'closure' is mentioned consider it an anonymous inner class.

In an ordinary method call --- that is, _not_ the sort of asynchronous, parallel calls typical in reactive programming --- the flow is something like this:

1) Call a method.
2) Store the return value from that method in a variable.
3) Use that variable and its new value to do something useful.

Or, something like this:

```groovy
returnVal = someMethod( *itsParameters* ); ⇒ make the call, assign its return value to `returnVal`
*do something useful with* returnVal;
```

In the asynchronous model the flow goes more like this:

1) Define a closure that does something useful with the return value from the asynchronous call, this is called the _observer_.
2) Define the asynchronous call itself as an object that inherits from [Observable|http://netflix.github.com/RxJava/rx/observables/Observable.html].
3) Attach the observer to that Observable asynchronous call by subscribing it (this also initiates the call).
4) Go on with your business; whenever the call returns, the _observer_ will begin to operate on its return value.

Which looks something like this:

```groovy
def myObserver = \{ it \-> *do something useful with* it \}; ⇒ defines, but does not invoke, the observer
def myObservable = *someObservableMethod*( *itsParameters* ); ⇒ defines, but does not invoke, the Observable
myObservable.subscribe(\[ onNext:myObserver \]); ⇒ subscribes the observer to the Observable, and invokes the Observable
*go on about my business*
```

## `onNext`, `onCompleted`, and `onError`

You will notice that the `subscribe( )` method does not take a simple closure as its parameter, but a Map. In the example above, this maps "`onNext`" onto the closure defined as `myObserver`.

`onNext` is a keyword particular to `subscribe()`. It defines the closure that the Observable will invoke whenever the Observable emits a value.

You can also pass in to the `subscribe()` method two additional closures that the Observable will invoke at different times:

| `onCompleted` | A Observable will invoke this closure after it has called `onNext` for the final time if it has not encountered any errors. |
| `onError^ | A Observable will invoke this closure to indicate that it has failed to generate the expected data. By default this stops the Observable and it will not make further calls to `onNext` or `onCompleted`. |

A more complete `subscribe()` example would therefore look like this:

```groovy
def myObserver   = \{ it \-> *do something useful with* it \};
def myComplete  = \{ *clean up after the final response* \};
def myError     = \{ exception \-> *react sensibly to a failed call* \};
def myObservable = *someMethod*( *itsParameters* );
myObservable.subscribe(\[ onNext:myObserver, onCompleted:myComplete, onError:myError \]);
*go on about my business*
```

# Creating Observables

When you use the various Observable utility functions to chain Observables together, you will occasionally want to convert other objects into Observables so that they can join in the chain. The methods explained in this section are simple ways to accomplish this.

## Making any iterable into a Observable --- `Observable.toObservable( )`

[[images/operation-toObservable.png]]

Any object that supports the `Iterable<>` interface can be converted into a Observable that emits each iterable item in the object, simply by passing the object into the `Observable.toObservable( )` method, for example:

```groovy
myObservable = Observable.toObservable( myIterable );
```

You can also do this with arrays, for example:

```groovy
myArray = [ 1, 2, 3, 4, 5 ];
myArrayObservable = Observable.toObservable( myArray );
```

This converts the sequence of values in the iterable object or array into a sequence of objects emitted, one at a time, by a Observable.

## Making any object into a Observable --- `Observable.just( )`

[[images/marble.just.png]]

To convert any object into a Observable that emits that object, pass that object into the `Observable.just()` method.

```groovy
def observableThatEmitsAString = Observable.just( "some string" ); // Observable emits "some string" as a single item
def observableThatEmitsAList = Observable.just( [1, 2, 3, 4, 5] ); // Observable emits the list [1, 2, 3, 4, 5] as a single item
```

This is similar to the `Observable.toObservable( )` method, except that `Observable.toObservable( )` will convert an iterable object into a Observable that emits each of the items in the iterable, one at a time, while the `Observable.just( )` method would convert the iterable into a Observable that emits the entire iterable as a single item.

## Creating a Observable --- `Observable.create( )`

You can create a simple Observable from scratch, by using the `Observable.create( )`|http://go/apidoc/com/netflix/api/platform/reactive/operations/ObservableExtensions.html#create(com.netflix.api.platform.functions.Func1)]^ method. You pass this method a closure that accepts as a parameter the map of closures that a observer passes to a Observable’s `subscribe( )` method. Write the closure you pass to `Observable.create( )` so that it behaves as a Observable --- calling the passed-in `onNext`, `onError`, and `onCompleted` methods appropriately. For example:

```groovy
def myObservable = Observable.create({ m ->
  m.onNext('One');
  m.onNext('Two');
  m.onNext('Three');
  m.onNext('Four');
  m.onCompleted();
})
```
{note}A well-formed Observable _must_ call either the observer’s `onCompleted( )` method exactly once or its `onError( )` method exactly once.{note}

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
