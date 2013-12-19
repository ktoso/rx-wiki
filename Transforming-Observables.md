This section explains operators with which you can transform items that are emitted by an Observable.

* [**`map( )`**](Transforming-Observables#map) — transform the items emitted by an Observable by applying a function to each of them
* [**`mapWithIndex( )`**](Transforming-Observables#mapwithindex) — transform the items emitted by an Observable by applying a function to each of them that takes into account the index value of the item
* [**`mapMany( )` or `flatMap( )`**](Transforming-Observables#mapmany-or-flatmap-and-mapmanydelayerror) — transform the items emitted by an Observable into Observables, then flatten this into a single Observable
* [**`mapManyDelayError( )`**](Transforming-Observables#mapmany-or-flatmap-and-mapmanydelayerror) — transform the items emitted by an Observable into Observables, then flatten this into a single Observable, waiting to report errors until all error-free observables have a chance to complete
* [**`reduce( )` or `aggregate( )`**](Transforming-Observables#reduce-or-aggregate) — apply a function to each emitted item, sequentially, and emit only the final accumulated value
* [**`scan( )`**](Transforming-Observables#scan) — apply a function to each item emitted by an Observable, sequentially, and emit each successive value
* [**`groupBy( )` and `groupByUntil( )`**](Transforming-Observables#groupby-and-groupbyuntil) — divide an Observable into a set of Observables that emit groups of items from the original Observable, organized by key
* [**`buffer( )`**](Transforming-Observables#buffer) — periodically gather items from an Observable into bundles and emit these bundles rather than emitting the items one at a time 
* [**`window( )`**](Transforming-Observables#window) — periodically subdivide items from an Observable into Observable windows and emit these windows rather than emitting the items one at a time 
* [**`cast( )`**](Transforming-Observables#cast) — cast all items from the source Observable into a particular type before reemitting them

***

## map( )
#### transform the items emitted by an Observable by applying a function to each of them
[[images/rx-operators/map.png]]

The `map( )` method applies a function of your choosing to every item emitted by an Observable, and returns this transformation as a new Observable. For example, the following code maps a function that squares the incoming value onto the values in `numbers`:

```groovy
numbers = Observable.from([1, 2, 3, 4, 5]);

numbers.map({it * it}).subscribe(
  { println(it); },                          // onNext
  { println("Error: " + it.getMessage()); }, // onError
  { println("Sequence complete"); }          // onCompleted
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

#### see also:
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#map(rx.util.functions.Func1)">`map(func)`</a>
* RxJS: <a href="https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md#rxobservableprototypemapselector-thisarg">`map`</a>
* Linq: <a href="http://msdn.microsoft.com/en-us/library/system.reactive.linq.observable.select.aspx">`Select`</a>
* <a href="http://www.introtorx.com/Content/v1.0.10621.0/08_Transformation.html#Select">Introduction to Rx: Select</a>

***

## mapWithIndex( )
#### transform the items emitted by an Observable by applying a function to each of them that takes into account the index value of the item
[[images/rx-operators/mapWithIndex.png]]

This version of `map( )` accepts a function that takes both the emitted item and the numerical index of that item in the sequence of emitted items as parameters, so that you can refer to both when you apply your transformation.

#### see also:
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#mapWithIndex(rx.util.functions.Func2)">`mapWithIndex(func)`</a>
* RxJS: <a href="https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md#rxobservableprototypeselectselector-thisarg">`select(selector,thisArg)`</a>
* Linq: <a href="http://msdn.microsoft.com/en-us/library/hh244311(v=vs.103).aspx">`Select(source, selector)`</a>

***

## mapMany( ) or flatMap( ), and mapManyDelayError( )
#### Transform the items emitted by an Observable into Observables, then flatten this into a single Observable
[[images/rx-operators/mapMany.png]]

The `mapMany( )` method (or `flatMap( )`, which has identical behavior) creates a new Observable by applying a function that you supply to each item emitted by the original Observable, where that function is itself an Observable that emits items, and then merges the results of that function applied to every item emitted by the original Observable, emitting these merged results.

This method is useful, for example, when you have an Observable that emits a series of items that themselves have Observable members or are in other ways transformable into Observables, so that you can create a new Observable that emits the complete collection of items emitted by the sub-Observables of these items.

```groovy
// this closure is an Observable that emits three numbers
numbers   = Observable.from([1, 2, 3]);
// this closure is an Observable that emits two numbers based on what number it is passed
multiples = { n -> Observable.from([ n*2, n*3 ]) };   

numbers.mapMany(multiples).subscribe(
  { println(it); },                          // onNext
  { println("Error: " + it.getMessage()); }, // onError
  { println("Sequence complete"); }          // onCompleted
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

#### see also:
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#mapMany(rx.util.functions.Func1)">`mapMany(func)`</a> (and <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#flatMap(rx.util.functions.Func1)">its `flatMap` clone</a>)
* RxJS: <a href="https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md#rxobservableprototypeselectmanyselector-resultselector">`selectMany`</a>
* Linq: <a href="http://msdn.microsoft.com/en-us/library/system.reactive.linq.observable.selectmany.aspx">`SelectMany`</a>
* <a href="http://www.introtorx.com/Content/v1.0.10621.0/08_Transformation.html#SelectMany">Introduction to Rx: SelectMany</a>

***

## reduce( ) or aggregate( )
#### Apply a function to each emitted item, sequentially, and emit only the final accumulated value
[[images/rx-operators/reduce.png]]

The `reduce( )` method (or `aggregate( )`, which has the same behavior) returns an Observable that applies a function of your choosing to the first item emitted by a source Observable, then feeds the result of that function along with the second item emitted by the source Observable into the same function, then feeds the result of _that_ function along with the third item into the same function, and so on until all items have been emitted by the source Observable. Then it emits the final result from the final call to your function as the sole output from the returned Observable.

Note that if the source Observable does not emit any items, `reduce( )` will fail with an `IllegalArgumentException`.

This technique, which is called “reduce” or “aggregate” in the RxJava context, is sometimes called “fold,” “accumulate,” “compress,” or “inject” in other programming arenas. 

For example, the following code uses `reduce( )` to compute, and then emit as an Observable, the sum of the numbers emitted by the source Observable:

```groovy
numbers = Observable.from([1, 2, 3, 4, 5]);

numbers.reduce({ a, b -> a+b }).subscribe(
  { println(it); },                          // onNext
  { println("Error: " + it.getMessage()); }, // onError
  { println("Sequence complete"); }          // onCompleted
);
```
```
15
Sequence complete
```

There is also a version of `reduce( )` to which you can pass a seed item in addition to an accumulator function:

```groovy
my_observable.reduce(initial_seed, accumulator_closure)
```

Note that passing a `null` seed is not the same as not passing a seed. The behavior will be different. If you pass a seed of `null`, you will be seeding your reduction with the item `null`. Note also that if you do pass in a seed, and the source Observable emits no items, `reduce` will emit the seed and complete normally without error.

#### example:

Imagine you have access to an Observable that emits a sequence of "Movie" objects that correspond to the "coming soon" movies from a theater. These objects include a number of items of information about the movie, including its title and opening day. You could use `reduce` to convert this sequence of Movie objects into a single list of titles, like this:

```groovy
getComingSoonSequence()
    .reduce([], { theList, video ->
                  theList.add("'" + video.getTitle() + "' (" + video.getOpen() + ")");
                  return(theList);
    }).subscribe({ println("Coming Soon: " + it) });
```
Which might result in something like this:
```
Coming Soon: ['Botso' (Sept. 30), 'The Act of Killing' (Sept. 30), 'Europa Report' (Sept. 27), 'Salinger' (Sept.27), 'In a World' (Sept. 27)]
```

#### see also:
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#reduce(rx.util.functions.Func2)">`reduce(accumulator)`</a> (and <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#aggregate(rx.util.functions.Func2)">its `aggregate` clone</a>)
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#reduce(R, rx.util.functions.Func2)">`reduce(initialValue, accumulator)`</a> (and <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#aggregate(R, rx.util.functions.Func2)">its `aggregate` clone</a>)
* RxJS: <a href="https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md#rxobservableprototypeaggregateseed-accumulator">`aggregate`</a>
* Linq: <a href="http://msdn.microsoft.com/en-us/library/system.reactive.linq.observable.aggregate.aspx">`Aggregate`</a>
* <a href="http://www.introtorx.com/Content/v1.0.10621.0/07_Aggregation.html#Aggregate">Introduction to Rx: Aggregate</a>

***

## scan( )
#### Apply a function to each item emitted by an Observable and emit each successive value
[[images/rx-operators/scan.png]]

The `scan( )` method returns an Observable that applies a function of your choosing to the first item emitted by a source Observable, then feeds the result of that function along with the second item emitted by the source Observable into the same function, then feeds the result of that function along with the third item into the same function, and so on until all items have been emitted by the source Observable. It emits the result of each of these iterations from the returned Observable. This sort of function is sometimes called an “accumulator.”

For example, the following code takes an Observable that emits a consecutive sequence of _n_ integers starting with 1 and converts it into an Observable that emits the first _n_ [triangular numbers](http://en.wikipedia.org/wiki/Triangular_number):

```groovy
numbers = Observable.from([1, 2, 3, 4, 5]);

numbers.scan({ a, b -> a+b }).subscribe(
  { println(it); },                          // onNext
  { println("Error: " + it.getMessage()); }, // onError
  { println("Sequence complete"); }          // onCompleted
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

There is also a version of `scan( )` to which you can pass a seed item in addition to an accumulator function:

```groovy
my_observable.scan(initial_seed, accumulator_closure)
```

**Note:** if you pass a seed item to `scan( )`, it will emit the seed itself as its first item.

Note also that passing a `null` seed is not the same as not passing a seed. The behavior will be different. If you pass a seed of `null`, you will be seeding your scan with `null`, and `scan( )` will emit `null` as its first item.

#### see also:
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#scan(rx.util.functions.Func2)">`scan(accumulator)`</a>
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#scan(R, rx.util.functions.Func2)">`scan(initialValue, accumulator)`</a>
* RxJS: <a href="https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md#rxobservableprototypescanseed-accumulator">`scan`</a>
* Linq: <a href="http://msdn.microsoft.com/en-us/library/system.reactive.linq.observable.scan.aspx">`Scan`</a>
* <a href="http://www.introtorx.com/Content/v1.0.10621.0/07_Aggregation.html#Scan">Introduction to Rx: Scan</a>

***

## groupBy( ) and groupByUntil( )
#### divide an Observable into a set of Observables that emit groups of items from the original Observable, organized by key
[[images/rx-operators/groupBy.png]]

The `groupBy( )` method creates or extracts a key from all of the items emitted by a source Observable. For each unique key created in this way, `groupBy( )` creates a [`GroupedObservable`](http://netflix.github.io/RxJava/javadoc/rx/observables/GroupedObservable.html) that emits all of the items emitted by the source Observable that match that key. `groupBy( )` then emits each of these Observables, as an Observable. A `GroupedObservable` has a method, [`getKey( )`](http://netflix.github.io/RxJava/javadoc/rx/observables/GroupedObservable.html#getKey()) with which you can retrieve the key that defines the `GroupedObservable`.

There are two versions of `groupBy( )`:

1. One version takes two parameters: the source Observable and a function that takes as its parameter an item emitted by the source Observable and returns the key.
1. The second version adds a third parameter: a function that takes as its parameter an item emitted by the source Observable and returns an item to be emitted by the resulting GroupedObservable (the first version just emits the source Observable's items unchanged).

The following sample code uses `groupBy( )` to transform a list of numbers into two lists, grouped by whether or not the numbers are even:
```groovy
def numbers = Observable.from([1, 2, 3, 4, 5, 6, 7, 8, 9]);
def groupFunc = { return(0 == (it % 2)); };

numbers.groupBy(groupFunc).mapMany({ it.reduce([it.getKey()], {a, b -> a << b}) }).subscribe(
  { println(it); },                          // onNext
  { println("Error: " + it.getMessage()); }, // onError
  { println("Sequence complete"); }          // onCompleted
);
```
```
[false, 1, 3, 5, 7, 9]
[true, 2, 4, 6, 8]
Sequence complete
```

There is also a `groupByUntil( )` operator. It has the two varieties mentioned above, but adds a third parameter: an Observable that emits duration markers. When a duration marker is emitted by this Observable, any grouped Observables that have been opened are closed, and `groupByUntil( )` will create new grouped Observables for any subsequent emissions by the source Observable.

[[images/rx-operators/groupByUntil.png]]

Another variety of `groupByUntil( )` limits the number of groups that can be active at any particular time. If an item is emitted by the source Observable that would cause the number of groups to exceed this maximum, before the new group is emitted, one of the existing groups is closed (that is, the Observable it represents terminates by calling its observers' `onCompleted` methods and then expires).

Note that when `groupBy( )` or `groupByUntil( )` splits up the source Observable into an Observable that emits Observables, it begins to emit items from the source Observable onto these emitted Observables immediately. That is to say, it does not wait for any Observers to subscribe. So if you want to ensure that you see all of the items that are emitted on these new Observables, you should take care to subscribe to them right away.

The following illustration shows how this can cause unexpected behavior:
[[images/rx-operators/groupBy.anomaly.png]]
In this illustration, `groupBy( )` is used to separate a source Observable that emits the numbers 1 through 6 into an Observable (in red) that emits two Observables: one that emits the odd numbers from the source Observable, and the other that emits the even numbers.  Then, this Observable of Observables, shown in red, is zipped with another Observable (shown in blue) that emits the strings "odd" and "even", and in the zip function, it applies this string label to all of the items emitted by the associated Observable emitted by the Observable shown in red.

However, `zip( )` does not apply this zip function until it observes an item emitted by each of the red and the blue source Observables.  Since "odd" arrives after the first of these Observables has already emitted "1", when `zip( )` has not yet subscribed to this Observable, it never observes this "1" and does not apply the zip function to it.  A similar thing happens with "even", which arrives after both "2" and "4" are emitted by the Observable emission it is paired with from the red-colored Observable.  For this reason, the transformed Observable emitted by `zip( )` is missing some of the data from the original Observables.

#### see also:
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#groupBy(rx.util.functions.Func1)">`groupBy(keySelector)`</a>
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#groupBy(rx.util.functions.Func1, rx.util.functions.Func1)">`groupBy(keySelector, elementSelector)`</a>
* RxJS: <a href="https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md#rxobservableprototypegroupbykeyselector-elementselector-keyserializer">`groupBy`</a>
* RxJS: <a href="https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md#rxobservableprototypegroupbyuntilkeyselector-elementselector-durationselector-keyserializer">`groupByUntil`</a>
* Linq: <a href="http://msdn.microsoft.com/en-us/library/system.reactive.linq.observable.groupby.aspx">`GroupBy`</a>
* Linq: <a href="http://msdn.microsoft.com/en-us/library/system.reactive.linq.observable.groupbyuntil.aspx">`GroupByUntil`</a>
* <a href="http://www.introtorx.com/Content/v1.0.10621.0/07_Aggregation.html#GroupBy">Introduction to Rx: GroupBy</a>

***

## buffer( )
#### periodically gather items emitted by an Observable into bundles and emit these bundles rather than emitting the items one at a time 
[[images/rx-operators/buffer.png]]

The `buffer( )` method periodically gathers items emitted by a source `Observable` into bundles, and emits these bundles as its own emissions. There are a number of ways with which you can regulate how `buffer( )` gathers items from the source `Observable` into bundles:

* `buffer(bufferOpenings, closingSelector)`
[[images/rx-operators/buffer2.png]]
> This version of `buffer( )` monitors an `Observable`, *bufferOpenings*, that emits `BufferOpening` objects. Each time it observes such an emitted object, it creates a new bundle to begin collecting items emitted by the source `Observable` and it passes the *bufferOpenings* `Observable` into the *closingSelector* function. That function returns an `Observable`. `buffer( )` monitors that `Observable` and when it detects an emitted object, it closes its bundle and emits it as its own emission.

* `buffer(count)`
[[images/rx-operators/buffer3.png]]
> This version of `buffer( )` emits a new bundle of items for every *count* items emitted by the source `Observable`.

* `buffer(count, skip)`
[[images/rx-operators/buffer4.png]]
> This version of `buffer( )` create a new bundle of items for every *skip* item(s) emitted by the source `Observable`, each containing *count* elements. If *skip* is less than *count* this means that the bundles will overlap and contain duplicate items. For example: `from([1, 2, 3, 4, 5]).buffer(3, 1)` will emit the following bundles: `[1, 2, 3]`, `[2, 3, 4]`, `[3, 4, 5]`.

* `buffer(timespan)` and `buffer(timespan, scheduler)`
[[images/rx-operators/buffer5.png]]
> This version of `buffer( )` emits a new bundle of items periodically, every *timespan* amount of time, containing all items emitted by the source `Observable` since the previous bundle emission.

* `buffer(timespan, count)` and `buffer(timespan, count, scheduler)`
[[images/rx-operators/buffer6.png]]
> This version of `buffer( )` emits a new bundle of items for every *count* items emitted by the source `Observable`, or, if *timespan* has elapsed since its last bundle emission, it emits a bundle of however many items the source `Observable` has emitted in that span, even if this is less than *count*.

* `buffer(timespan, timeshift)` and `buffer(timespan, timeshift, scheduler)`
[[images/rx-operators/buffer7.png]]
> This version of `buffer( )` creates a new bundle of items every *timeshift*, and fills this bundle with every item emitted by the source `Observable` from that time until *timespan* time has passed since the bundle's creation, before emitting the bundle as its own emission. If *timespan* is longer than *timeshift*, the emitted bundles will represent time periods that overlap and so they may contain duplicate items.

#### see also:
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#buffer(rx.util.functions.Func0)">`buffer(closingSelector)`</a>
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#buffer(int)">`buffer(count)`</a>
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#buffer(int, int)">`buffer(count, skip)`</a>
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#buffer(long, long, java.util.concurrent.TimeUnit)">`buffer(timespan, timeshift, unit)`</a>
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#buffer(long, long, java.util.concurrent.TimeUnit, rx.Scheduler)">`buffer(timespan, timeshift, unit, scheduler)`</a>
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#buffer(long, java.util.concurrent.TimeUnit)">`buffer(timespan, unit)`</a>
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#buffer(long, java.util.concurrent.TimeUnit, int)">`buffer(timespan, unit, count)`</a>
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#buffer(long, java.util.concurrent.TimeUnit, int, rx.Scheduler)">`buffer(timespan, unit, count, scheduler)`</a>
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#buffer(long, java.util.concurrent.TimeUnit, rx.Scheduler)">`buffer(timespan, unit, scheduler)`</a>
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#buffer(rx.Observable, rx.util.functions.Func1)">`buffer(bufferOpenings, closingSelector)`</a>
* RxJS: <a href="https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md#rxobservableprototypebufferbufferopenings-bufferboundaries-bufferclosingselector">`buffer`</a>, <a href="https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md#rxobservableprototypebufferwithcountcount-skip">`bufferWithCount`</a>, <a href="https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md#rxobservableprototypebufferwithtimetimespan-timeshift--scheduler-scheduler">`bufferWithTime`</a>, and <a href="https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md#rxobservableprototypebufferwithtimeorcounttimespan-count-scheduler">`bufferWithTimeOrCount`</a>
* Linq: <a href="http://msdn.microsoft.com/en-us/library/system.reactive.linq.observable.buffer.aspx">`Buffer`</a>
* <a href="http://www.introtorx.com/Content/v1.0.10621.0/13_TimeShiftedSequences.html#Buffer">Introduction to Rx: Buffer</a>
* <a href="http://www.introtorx.com/Content/v1.0.10621.0/17_SequencesOfCoincidence.html#BufferRevisted">Introduction to Rx: Buffer revisited</a>

***

## window( )
#### periodically subdivide items from an Observable into Observable windows and emit these windows rather than emitting the items one at a time 
[[images/rx-operators/window.png]]

Window is similar to `buffer( )`, but rather than emitting packets of items from the original `Observable`, it emits `Observable`s, each one of which emits a subset of items from the original `Observable` and then terminates with an `onCompleted( )` call.

Like `buffer( )`, `window( )` has many varieties, each with its own way of subdividing the original `Observable` into the resulting `Observable` emissions, each one of which contains a "window" onto the original emitted items. In the terminology of the `window( )` method, when a window "opens," this means that a new `Observable` is emitted and that `Observable` will begin emitting items emitted by the source `Observable`. When a window "closes," this means that the emitted `Observable` stops emitting items from the source `Observable` and calls its Observers' `onCompleted( )` method and terminates.

* `window(source, closingSelector)`
[[images/rx-operators/window1.png]]
> This version of `window( )` opens its first window immediately. It closes the currently open window and immediately opens a new one each time it observes an object emitted by the `Observable` that is returned from *closingSelector*. In this way, this version of `window( )` emits a series of non-overlapping windows whose collective `onNext( )` emissions correspond one-to-one with those of the *source* `Observable`.

* `window(source, windowOpenings, closingSelector)`
[[images/rx-operators/window2.png]]
> This version of `window( )` opens a window whenever it observes the *windowOpenings* `Observable` emit an `Opening` object and at the same time calls *closingSelector* to generate a closing `Observable` associated with that window. When that closing `Observable` emits an object, `window( )` closes that window. Since the closing of currently open windows and the opening of new windows are activities that are regulated by independent `Observable`s, this version of `window( )` may create windows that overlap (duplicating items from the *source* `Observable`) or that leave gaps (discarding items from the *source* `Observable`).

* `window(source, count)`
[[images/rx-operators/window3.png]]
> This version of `window( )` opens its first window immediately. It closes the currently open window and immediately opens a new one whenever the current window has emitted *count* items. It will also close the currently open window if it receives an `onCompleted( )` or `onError( )` call from the *source* `Observable`. This version of `window( )` emits a series of non-overlapping windows whose collective `onNext( )` emissions correspond one-to-one with those of the *source* `Observable`.

* `window(source, count, skip)`
[[images/rx-operators/window4.png]]
> This version of `window( )` opens its first window immediately. It opens a new window beginning with every *skip* item from the source `Observable` (e.g. if *skip* is 3, then it opens a new window starting with every third item). It closes each window when that window has emitted *count* items or if it receives an `onCompleted( )` or `onError( )` call from the *source* `Observable`. If *skip* = *count* then this behaves the same as `window(source, count)`; if *skip* < *count* this will emit windows that overlap by *count* - *skip* items; if *skip* > *count* this will emit windows that drop *skip* - *count* items from the *source* `Observable` between every window.

* `window(source, timespan, unit)` and `window(source, timespan, unit, scheduler)`
[[images/rx-operators/window5.png]]
> This version of `window( )` opens its first window immediately. It closes the currently open window and opens another one every *timespan* period of time (measured in *unit*, and optionally on a particular *scheduler*). It will also close the currently open window if it receives an `onCompleted( )` or `onError( )` call from the *source* `Observable`. This version of `window( )` emits a series of non-overlapping windows whose collective `onNext( )` emissions correspond one-to-one with those of the *source* `Observable`.

* `window(source, timespan, unit, count)` and `window(source, timespan, unit, count, scheduler)`
[[images/rx-operators/window6.png]]
> This version of `window( )` opens its first window immediately. It closes the currently open window and opens another one every *timespan* period of time (measured in *unit*, and optionally on a particular *scheduler*) or whenever the currently open window has emitted *count* items. It will also close the currently open window if it receives an `onCompleted( )` or `onError( )` call from the *source* `Observable`. This version of `window( )` emits a series of non-overlapping windows whose collective `onNext( )` emissions correspond one-to-one with those of the *source* `Observable`.

* `window(source, timespan, timeshift, unit)` and `window(source, timespan, timeshift, unit, scheduler)`
[[images/rx-operators/window7.png]]
> This version of `window( )` opens its first window immediately, and thereafter opens a new window every *timeshift* period of time (measured in *unit*, and optionally on a particular *scheduler*). It closes a currently open window after *timespan* period of time has passed since that window was opened. It will also close any currently open window if it receives an `onCompleted( )` or `onError( )` call from the *source* `Observable`. Depending on how you set *timespan* and *timeshift* the windows that result from this operation may overlap or have gaps.

#### see also:
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#window(rx.util.functions.Func0)">`window(closingSelector)`</a>
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#window(int)">`window(count)`</a>
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#window(int, int)">`window(count, skip)`</a>
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#window(long, long, java.util.concurrent.TimeUnit)">`window(timespan, timeshift, unit)`</a>
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#window(long, long, java.util.concurrent.TimeUnit, rx.Scheduler)">`window(timespan, timeshift, unit, scheduler)`</a>
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#window(long, java.util.concurrent.TimeUnit)">`window(timespan, unit)`</a>
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#window(long, java.util.concurrent.TimeUnit, int)">`window(timespan, unit, count)`</a>
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#window(long, java.util.concurrent.TimeUnit, int, rx.Scheduler)">`window(timespan, unit, count, scheduler)`</a>
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#window(long, java.util.concurrent.TimeUnit, rx.Scheduler)">`window(timespan, unit, scheduler)`</a>
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#window(rx.Observable, rx.util.functions.Func1)">`window(windowOpenings, closingSelector)`</a>
* RxJS: <a href="https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md#rxobservableprototypewindowwindowopenings-windowboundaries-windowclosingselector">`window`</a>, <a href="https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md#rxobservableprototypewindowwithcountcount-skip">`windowWithCount`</a>, <a href="https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md#rxobservableprototypewindowwithtimetimespan-timeshift--scheduler">`windowWithTime`</a>, and <a href="https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md#rxobservableprototypewindowwithtimeorcounttimespan-count-scheduler">`windowWithTimeOrCount`</a>
* Linq: <a href="http://msdn.microsoft.com/en-us/library/system.reactive.linq.observable.window.aspx">`Window`</a>
* <a href="http://www.introtorx.com/Content/v1.0.10621.0/17_SequencesOfCoincidence.html#Window">Introduction to Rx: Window</a>

***

## cast( )
#### cast all items from the source Observable into a particular type before reemitting them
[[images/rx-operators/cast.png]]

#### see also:
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#cast(java.lang.Class)">`cast(class)`</a>
* Linq: <a href="http://msdn.microsoft.com/en-us/library/hh211842(v=vs.103).aspx">`Cast`</a>
* <a href="http://www.introtorx.com/Content/v1.0.10621.0/08_Transformation.html#CastAndOfType">Introduction to Rx: Cast and OfType</a>
