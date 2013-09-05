This section explains operators with which you can transform items that are emitted by an Observable.

* [**`map( )`**](Transforming-Observables#map) — transform the items emitted by an Observable by applying a function to each of them
* [**`mapMany( )` or `flatMap( )`**](Transforming-Observables#mapmany-or-flatmap-and-mapmanydelayerror) — transform the items emitted by an Observable into Observables, then flatten this into a single Observable
* [**`mapManyDelayError( )`**](Transforming-Observables#mapmany-or-flatmap-and-mapmanydelayerror) — transform the items emitted by an Observable into Observables, then flatten this into a single Observable, waiting to report errors until all error-free observables have a chance to complete
* [**`reduce( )` or `aggregate( )`**](Transforming-Observables#reduce-or-aggregate) — apply a function to each emitted item, sequentially, and emit only the final accumulated value
* [**`scan( )`**](Transforming-Observables#scan) — apply a function to each item emitted by an Observable, sequentially, and emit each successive value
* [**`groupBy( )`**](Transforming-Observables#groupby) — divide an Observable into a set of Observables that emit groups of items from the original Observable, organized by key
* [**`buffer( )`**](Transforming-Observables#buffer) — periodically gather items from an Observable into bundles and emit these bundles rather than emitting the items one at a time 
* [**`window( )`**](Transforming-Observables#window) — periodically subdivide items from an Observable into Observable windows and emit these windows rather than emitting the items one at a time 

## map( )
#### transform the items emitted by an Observable by applying a function to each of them
[[images/rx-operators/map.png]]

The `map( )` method applies a function of your choosing to every item emitted by an Observable, and returns this transformation as a new Observable. For example, the following code maps a function that squares the incoming value onto the values in `numbers`:

```groovy
numbers = Observable.toObservable([1, 2, 3, 4, 5]);

Observable.map(numbers, {it * it}).subscribe(
  [ onNext:{ myWriter.println(it); },
    onCompleted:{ myWriter.println("Sequence complete"); },
    onError:{ myWriter.println("Error encountered"); } ]
);
```
```
1
4
9
16
25
Sequence complete
```

In addition to calling `map( )` as a stand-alone method, you can also call it as a method of an Observable, so, in the example above, instead of 

```groovy
Observable.map(numbers, { it * it }) ...
```

you could instead write 

```groovy
numbers.map({ it * it }) ...
```

## mapMany( ) or flatMap( ), and mapManyDelayError( )
#### Transform the items emitted by an Observable into Observables, then flatten this into a single Observable
[[images/rx-operators/mapMany.png]]

The `mapMany( )` method (or `flatMap( )`, which has identical behavior) creates a new Observable by applying a function that you supply to each item emitted by the original Observable, where that function is itself an Observable that emits items, and then merges the results of that function applied to every item emitted by the original Observable, emitting these merged results.

This method is useful, for example, when you have an Observable that emits a series of items that themselves have Observable members or are in other ways transformable into Observables, so that you can create a new Observable that emits the complete collection of items emitted by the sub-Observables of these items.

```groovy
// this closure is an Observable that emits three numbers
numbers   = Observable.toObservable([1, 2, 3]);
// this closure is an Observable that emits two numbers based on what number it is passed
multiples = { n -> Observable.toObservable([ n*2, n*3 ]) };   

numbers.mapMany(multiples).subscribe(
  [ onNext:{ myWriter.println(it.toString()); },
    onCompleted:{ myWriter.println("Sequence complete"); },
    onError:{ myWriter.println("Error encountered"); } ]
);
```
```
2
3
4
6
6
9
Sequence complete
```

If any of the individual Observables mapped to the items from the source Observable in `mapMany( )` aborts by invoking `onError`, the `mapMany( )` call itself will immediately abort and invoke `onError`. If you would prefer that the map-many operation continue emitting the results of the remaining, error-free Observables before reporting the error, use `mapManyDelayError( )` instead.

[[images/rx-operators/mapManyDelayError.png]]

Because it is possible for more than one of the individual Observables to encounter an error, `mapManyDelayError( )` may pass information about multiple errors to the `onError` method of its Observers (which it will never invoke more than once). For this reason, if you want to know the nature of these errors, you should write your `onError` method so that it accepts a parameter of the class [`CompositeException`](http://netflix.github.io/RxJava/javadoc/rx/util/CompositeException.html).

## reduce( ) or aggregate( )
#### Apply a function to each emitted item, sequentially, and emit only the final accumulated value
[[images/rx-operators/reduce.png]]

The `reduce( )` method (or `aggregate( )`, which has the same behavior) returns an Observable that applies a function of your choosing to the first item emitted by a source Observable, then feeds the result of that function along with the second item emitted by the source Observable into the same function, then feeds the result of _that_ function along with the third item into the same function, and so on until all items have been emitted by the source Observable. Then it emits the final result from the final call to your function as the sole output from the returned Observable.

This technique, which is called “reduce” or “aggregate” here, is sometimes called “fold,” “accumulate,” “compress,” or “inject” in other programming contexts. 

For example, the following code uses `reduce( )` to compute, and then emit as an Observable, the sum of the numbers emitted by the source Observable:

```groovy
numbers = Observable.toObservable([1, 2, 3, 4, 5]);

Observable.reduce(numbers, { a, b -> a+b }).subscribe(
  [ onNext:{ myWriter.println(it); },
    onCompleted:{ myWriter.println("Sequence complete"); },
    onError:{ myWriter.println("Error encountered"); } ]
);
```
```
15
Sequence complete
```

In addition to calling `reduce( )` as a stand-alone method, you can also call it as a method of an Observable, so, in the example above, instead of 

```groovy
Observable.reduce(numbers, { a, b -> a+b }) ...
```
you could instead write 

```groovy
numbers.reduce({ a, b -> a+b }) ...
```

There is also a version of `reduce( )` to which you can pass a seed item in addition to an accumulator function:

```groovy
Observable.reduce(my_observable, initial_seed, accumulator_closure)
```
or
```groovy
my_observable.reduce(initial_seed, accumulator_closure)
```

Note that passing a `null` seed is not the same as not passing a seed. The behavior will be different. If you pass a seed of `null`, you will be seeding your reduction with the item `null`.

## scan( )
#### Apply a function to each item emitted by an Observable and emit each successive value
[[images/rx-operators/scan.png]]

The `scan( )` method returns an Observable that applies a function of your choosing to the first item emitted by a source Observable, then feeds the result of that function along with the second item emitted by the source Observable into the same function, then feeds the result of that function along with the third item into the same function, and so on until all items have been emitted by the source Observable. It emits the result of each of these iterations from the returned Observable. This sort of function is sometimes called an “accumulator.”

For example, the following code takes an Observable that emits a consecutive sequence of _n_ integers starting with 1 and converts it into an Observable that emits the first _n_ [triangular numbers](http://en.wikipedia.org/wiki/Triangular_number):

```groovy
numbers = Observable.toObservable([1, 2, 3, 4, 5]);

Observable.scan(numbers, { a, b -> a+b }).subscribe(
  [ onNext:{ myWriter.println(it); },
    onCompleted:{ myWriter.println("Sequence complete"); },
    onError:{ myWriter.println("Error encountered"); } ]
);
```
```
1
3
6
10
15
Sequence complete
```

In addition to calling `scan( )` as a stand-alone method, you can also call it as a method of an Observable, so, in the example above, instead of 

```groovy
Observable.scan(numbers, { a, b -> a+b }) ...
```
you could instead write 
```groovy
numbers.scan({ a, b -> a+b }) ...
```

There is also a version of `scan( )` to which you can pass a seed item in addition to an accumulator function:

```groovy
Observable.scan(my_observable, initial_seed, accumulator_closure)
```
or
```groovy
my_observable.scan(initial_seed, accumulator_closure)
```

**Note:** if you pass a seed item to `scan( )`, it will emit the seed itself as its first item.

Note also that passing a `null` seed is not the same as not passing a seed. The behavior will be different. If you pass a seed of `null`, you will be seeding your scan with `null`, and `scan( )` will emit `null` as its first item.

## groupBy( )
#### divide an Observable into a set of Observables that emit groups of items from the original Observable, organized by key
[[images/rx-operators/groupBy.png]]

The `groupBy( )` method creates or extracts a key from all of the items emitted by a source Observable. For each unique key created in this way, `groupBy( )` creates a [`GroupedObservable`](http://netflix.github.io/RxJava/javadoc/rx/observables/GroupedObservable.html) that emits all of the items emitted by the source Observable that match that key. `groupBy( )` then emits each of these Observables, as an Observable. A `GroupedObservable` has a method, [`getKey( )`](http://netflix.github.io/RxJava/javadoc/rx/observables/GroupedObservable.html#getKey()) with which you can retrieve the key that defines the `GroupedObservable`.

There are two versions of `groupBy( )`:

1. One version takes two parameters: the source Observable and a function that takes as its parameter an item emitted by the source Observable and returns the key.
1. The second version adds a third parameter: a function that takes as its parameter an item emitted by the source Observable and returns an item to be emitted by the resulting GroupedObservable (the first version just emits the source Observable's items unchanged).

The following sample code uses `groupBy( )` to transform a list of numbers into two lists, grouped by whether or not the numbers are even:
```groovy
class isEven implements rx.util.functions.Func1
{
  java.lang.Object call(java.lang.Object T) { return(0 == (T % 2)); }
}

def numbers = Observable.toObservable([1, 2, 3, 4, 5, 6, 7, 8, 9]);
def groupFunc = new isEven();

numbers.groupBy(groupFunc).mapMany({ Observable.reduce(it, [it.getKey()], {a, b -> a << b}) }).subscribe(
  [onNext:{ api.servletResponse.getWriter().println(it) },
   onCompleted:{ api.servletResponse.getWriter().println("Sequence complete"); },
   onError:{ api.servletResponse.getWriter().println("Error encountered"); } ]
)
```
```
[false, 1, 3, 5, 7, 9]
[true, 2, 4, 6, 8]
Sequence complete
```

## buffer( )
#### periodically gather items emitted by an Observable into bundles and emit these bundles rather than emitting the items one at a time 
[[images/rx-operators/buffer.png]]

The `buffer( )` method periodically gathers items emitted by a source `Observable` into bundles, and emits these bundles as its own emissions. There are a number of ways with which you can regulate how `buffer( )` gathers items from the source `Observable` into bundles:

* `buffer(count)`

> This version of `buffer( )` emits a new bundle of items for every *count* items emitted by the source `Observable`.

* `buffer(timespan)` and `buffer(timespan, scheduler)`

> This version of `buffer( )` emits a new bundle of items periodically, every *timespan* amount of time, containing all items emitted by the source `Observable` since the previous bundle emission.

* `buffer(timespan, count)` and `buffer(timespan, count, scheduler)`

> This version of `buffer( )` emits a new bundle of items for every *count* items emitted by the source `Observable`, or, if *timespan* has elapsed since its last bundle emission, it emits a bundle of however many items the source `Observable` has emitted in that span, even if this is less than *count*.

* `buffer(count, skip)`

> This version of `buffer( )` create a new bundle of items for every *skip* item(s) emitted by the source `Observable`, each containing *count* elements. If *skip* is less than *count* this means that the bundles will overlap and contain duplicate items. For example: `toObservable([1, 2, 3, 4, 5]).buffer(3, 1)` will emit the following bundles: `[1, 2, 3]`, `[2, 3, 4]`, `[3, 4, 5]`.

* `buffer(timespan, timeshift)` and `buffer(timespan, timeshift, scheduler)`

> This version of `buffer( )` creates a new bundle of items every *timeshift*, and fills this bundle with every item emitted by the source `Observable` from that time until *timespan* time has passed since the bundle's creation, before emitting the bundle as its own emission. If *timespan* is longer than *timeshift*, the emitted bundles will represent time periods that overlap and so they may contain duplicate items.

* `buffer(bufferOpenings, bufferClosingSelector)`

> This version of `buffer( )` monitors an `Observable`, *bufferOpenings*, that emits `BufferOpening` objects. Each time it observes such an emitted object, it creates a new bundle to begin collecting items emitted by the source `Observable` and it passes the *bufferOpenings* `Observable` into the *bufferClosingSelector* function. That function returns an `Observable` that emits `BufferClosing` objects. `buffer( )` monitors that `Observable` and when it detects an emitted `BufferClosing` object, it closes its bundle and emits it as its own emission.

## window( )
#### periodically subdivide items from an Observable into Observable windows and emit these windows rather than emitting the items one at a time 
[[images/rx-operators/window1.png]]

Window is similar to `buffer( )`, but rather than emitting packets of items from the original `Observable`, it emits `Observable`s, each one of which emits a subset of items from the original `Observable` and then terminates with an `onCompleted( )` call.

Like `buffer( )`, `window( )` has many varieties, each with its own way of subdividing the original `Observable` into the resulting `Observable` emissions, each one of which contains a "window" onto the original emitted items. In the terminology of the `window( )` method, when a window "opens," this means that a new `Observable` is emitted and that `Observable` will begin emitting items emitted by the source `Observable`. When a window "closes," this means that the emitted `Observable` stops emitting items from the source `Observable` and calls its Observers' `onCompleted( )` method and terminates.

* `window(source, closingSelector)`
> This version of `window( )` opens its first window immediately. It closes the currently open window and immediately opens a new one each time it observes a `Closing` object emitted by the `Observable` that is returned from *closingSelector*. In this way, this version of `window( )` emits a series of non-overlapping windows whose collective `onNext( )` emissions correspond one-to-one with those of the *source* `Observable`.

* `window(source, windowOpenings, closingSelector)`
> This version of `window( )` opens a window whenever it observes the *windowOpenings* `Observable` emit an `Opening` object and at the same time calls *closingSelector* to generate a closing `Observable` associated with that window. When that closing `Observable` emits a `Closing` object, `window( )` closes that window. Since the closing of currently open windows and the opening of new windows are activities that are regulated by independent `Observable`s, this version of `window( )` may create windows that overlap (duplicating items from the *source* `Observable`) or that leave gaps (discarding items from the *source* `Observable`).

* `window(source, count)`
> This version of `window( )` opens its first window immediately. It closes the currently open window and immediately opens a new one whenever the current window has emitted *count* items. It will also close the currently open window if it receives an `onCompleted( )` or `onError( )` call from the *source* `Observable`. This version of `window( )` emits a series of non-overlapping windows whose collective `onNext( )` emissions correspond one-to-one with those of the *source* `Observable`.

* `window(source, count, skip)`
> This version of `window( )` opens its first window immediately. It opens a new window beginning with every *skip* item from the source `Observable` (e.g. if *skip* is 3, then it opens a new window starting with every third item). It closes each window when that window has emitted *count* items or if it receives an `onCompleted( )` or `onError( )` call from the *source* `Observable`. If *skip* = *count* then this behaves the same as `window(source, count)`; if *skip* < *count* this will emit windows that overlap by *count* - *skip* items; if *skip* > *count* this will emit windows that drop *skip* - *count* items from the *source* `Observable` between every window.

* `window(source, timespan, unit)` and `window(source, timespan, unit, scheduler)`
> This version of `window( )` opens its first window immediately. It closes the currently open window and opens another one every *timespan* period of time (measured in *unit*, and optionally on a particular *scheduler*). It will also close the currently open window if it receives an `onCompleted( )` or `onError( )` call from the *source* `Observable`. This version of `window( )` emits a series of non-overlapping windows whose collective `onNext( )` emissions correspond one-to-one with those of the *source* `Observable`.

* `window(source, timespan, unit, count)` and `window(source, timespan, unit, count, scheduler)`
> This version of `window( )` opens its first window immediately. It closes the currently open window and opens another one every *timespan* period of time (measured in *unit*, and optionally on a particular *scheduler*) or whenever the currently open window has emitted *count* items. It will also close the currently open window if it receives an `onCompleted( )` or `onError( )` call from the *source* `Observable`. This version of `window( )` emits a series of non-overlapping windows whose collective `onNext( )` emissions correspond one-to-one with those of the *source* `Observable`.

* `window(source, timespan, timeshift, unit)` and `window(source, timespan, timeshift, unit, scheduler)`
> This version of `window( )` opens its first window immediately, and thereafter opens a new window every *timeshift* period of time (measured in *unit*, and optionally on a particular *scheduler*). It closes a currently open window after *timespan* period of time has passed since that window was opened. It will also close any currently open window if it receives an `onCompleted( )` or `onError( )` call from the *source* `Observable`. Depending on how you set *timespan* and *timeshift* the windows that result from this operation may overlap or have gaps.
