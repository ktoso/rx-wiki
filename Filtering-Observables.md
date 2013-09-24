This section explains operators you can use to filter and select items emitted by Observables.

* [**`filter( )` or `where( )`**](Filtering-Observables#filter-or-where) — filter items emitted by an Observable
* [**`takeLast( )`**](Filtering-Observables#takelast) — only emit the last _n_ items emitted by an Observable
* [**`skip( )`**](Filtering-Observables#skip) — ignore the first _n_ items emitted by an Observable
* [**`skipWhile( )` and `skipWhileWithIndex( )`**](Filtering-Observables#skipwhile-and-skipwhilewithindex) — discard items emitted by an Observable until a specified condition is false, then emit the remainder
* [**`take( )`**](Filtering-Observables#take) — emit only the first _n_ items emitted by an Observable
* [**`takeWhile( )` and `takeWhileWithIndex( )`**](Filtering-Observables#takewhile-and-takewhilewithindex) — emit items emitted by an Observable as long as a specified condition is true, then skip the remainder
* [**`first( )`**](Filtering-Observables#first) — emit only the first item emitted by an Observable, or the first item that meets some condition
* [**`firstOrDefault( )`**](Filtering-Observables#firstordefault) — emit only the first item emitted by an Observable, or the first item that meets some condition, or a default value if the source Observable is empty
* [**`elementAt( )`**](Filtering-Observables#elementat) — emit item _n_ emitted by the source Observable
* [**`elementAtOrDefault( )`**](Filtering-Observables#elementatordefault) — emit item _n_ emitted by the source Observable, or a default item if the source Observable emits fewer than _n_ items
* [**`sample( )` or `throttleLast( )`**](Filtering-Observables#sample-or-throttlelast) — emit the most recent items emitted by an Observable within periodic time intervals
* [**`throttleFirst( )`**](Filtering-Observables#throttlefirst) — emit the first items emitted by an Observable within periodic time intervals
* [**`throttleWithTimeout( )` or `debounce( )`**](Filtering-Observables#throttlewithtimeout-or-debounce) — only emit an item from the source Observable after a particular timespan has passed without the Observable emitting any other items
* [**`distinct( )`**](Filtering-Observables#distinct) — suppress duplicate items emitted by the source Observable
* [**`distinctUntilChanged( )`**](Filtering-Observables#distinctuntilchanged) — suppress duplicate consecutive items emitted by the source Observable
* [**`ofClass( )`**](Filtering-Observables#ofclass) — emit only those items from the source Observable that are of a particular class

***

## filter( ) or where( )
#### filter items emitted by an Observable
[[images/rx-operators/filter.png]]

You can filter an Observable, discarding any items that do not meet some test, by passing a filtering function into the `filter( )` method. For example, the following code filters a list of integers, emitting only those that are even (that is, where the remainder from dividing the number by two is zero):

```groovy
numbers = Observable.from([1, 2, 3, 4, 5, 6, 7, 8, 9]);

Observable.filter(numbers, { 0 == (it % 2) }).subscribe(
  { println(it); },                  // onNext
  { println("Error encountered"); }, // onError
  { println("Sequence complete"); }  // onCompleted
);
```
```
2
4
6
8
Sequence complete
```

In addition to calling `filter( )` as a stand-alone method, you can also call it as a method of an Observable, so, in the example above, instead of 

```groovy
Observable.filter(numbers, { 0 == (it %2) }) ...
```
you could instead write 
```groovy
numbers.filter({ 0 == (it % 2) }) ...
```

The `where( )` method has the same purpose as `filter( )` but accepts a `Func1` evaluator function instead of a closure. Here is the same sample, but implemented with `where( )` instead of `filter( )`:
```groovy
class isEven implements rx.util.functions.Func1 {
  Boolean call( Object it ) { return(0 == (it % 2)); }
}

myisEven = new isEven();

numbers = Observable.from([1, 2, 3, 4, 5, 6, 7, 8, 9]);

numbers.where(myisEven).subscribe(
  { println(it); },                  // onNext
  { println("Error encountered"); }, // onError
  { println("Sequence complete"); }  // onCompleted
);
```

#### see also:
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#filter(rx.util.functions.Func1)">`filter(predicate)`</a> (and <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#where(rx.util.functions.Func1)">its `where` clone</a>)
* RxJS: <a href="https://github.com/Reactive-Extensions/RxJS/wiki/Observable#wiki-where">`where`</a>
* Linq: <a href="http://msdn.microsoft.com/en-us/library/system.reactive.linq.observable.where(v=vs.103).aspx">`Where`</a>
* <a href="http://www.introtorx.com/Content/v1.0.10621.0/05_Filtering.html#Where">Introduction to Rx: Where</a>

***

## takeLast( )
#### only emit the last _n_ items emitted by an Observable
[[images/rx-operators/last.png]]

To convert an Observable that emits several items into one that only emits the last _n_ of these itemsbefore completing, use the `takeLast( )` method. For instance, in the following code, `takeLast( )` emits only the last integer in the list of integers represented by `numbers`:

```groovy
numbers = Observable.from([1, 2, 3, 4, 5, 6, 7, 8, 9]);

Observable.takeLast(numbers,1).subscribe(
  { println(it); },                  // onNext
  { println("Error encountered"); }, // onError
  { println("Sequence complete"); }  // onCompleted
);
```
```
9
Sequence complete
```

In addition to calling `takeLast( )` as a stand-alone method, you can also call it as a method of an Observable, so, in the example above, instead of 

```groovy
Observable.takeLast(numbers,1) ...
```
you could instead write
```groovy
numbers.takeLast(1) ...
```

#### see also:
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#takeLast(int)">`takeLast(count)`</a>
* RxJS: <a href="https://github.com/Reactive-Extensions/RxJS/wiki/Observable#wiki-takeLast">`takeLast`</a>
* Linq: <a href="http://msdn.microsoft.com/en-us/library/hh212114(v=vs.103).aspx">`TakeLast`</a>
* <a href="http://www.introtorx.com/Content/v1.0.10621.0/07_Aggregation.html#Last">Introduction to Rx: Last</a>

***

## skip()
#### ignore the first _n_ items emitted by an Observable
[[images/rx-operators/skip.png]]

You can ignore the first _n_ items emitted by an Observable and attend only to those items that come after, by modifying the Observable with the `skip(n)` method.

```groovy
numbers = Observable.from([1, 2, 3, 4, 5, 6, 7, 8, 9]);

Observable.skip(numbers, 3).subscribe(
  { println(it); },                  // onNext
  { println("Error encountered"); }, // onError
  { println("Sequence complete"); }  // onCompleted
);
```
```
4
5
6
7
8
9
Sequence complete
```

In addition to calling `skip( )` as a stand-alone method, you can also call it as a method of an Observable, so, in the example above, instead of 

```groovy
Observable.skip(numbers, 3) ...
```
you could instead write 
```groovy
numbers.skip(3) ...
```

#### see also:
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#skip(int)">`skip(num)`</a>
* RxJS: <a href="https://github.com/Reactive-Extensions/RxJS/wiki/Observable#wiki-single">`skip`</a>
* Linq: <a href="http://msdn.microsoft.com/en-us/library/hh229847(v=vs.103).aspx">`Skip`</a>
* <a href="http://www.introtorx.com/Content/v1.0.10621.0/05_Filtering.html#SkipAndTake">Introduction to Rx: Skip and Take</a>

***

## skipWhile( ) and skipWhileWithIndex( )
#### discard items emitted by an Observable until a specified condition is false, then emit the remainder
[[images/rx-operators/skipWhile.png]]

The `skipWhile( )` method returns an Observable that discards items emitted by the source Observable until such time as a function applied to an item emitted by that Observable returns `false`, whereupon the new Observable emits that item and the remainder of the items emitted by the source Observable.

```groovy
numbers = Observable.from( [1, 2, 3, 4, 5, 6, 7, 8, 9] );

numbers.skipWhile({ (0 == (it % 5)) }).subscribe(
  { println(it); },                  // onNext
  { println("Error encountered"); }, // onError
  { println("Sequence complete"); }  // onCompleted
);
```
```
5
6
7
8
9
Sequence complete
```

[[images/rx-operators/skipWhileWithIndex.png]]

The `skipWhileWithIndex( )` method is similar, but your function takes an additional parameter: the (zero-based) index of the item being emitted by the source Observable.
```groovy
numbers = Observable.from( [1, 2, 3, 4, 5, 6, 7, 8, 9] );

numbers.skipWhileWithIndex({ it, index -> ((it < 6) || (index < 5)) }).subscribe(
  { println(it); },                  // onNext
  { println("Error encountered"); }, // onError
  { println("Sequence complete"); }  // onCompleted
);
```
```
6
7
8
9
Sequence complete
```

#### see also:
* RxJS: <a href="https://github.com/Reactive-Extensions/RxJS/wiki/Observable#wiki-skipWhile">`skipWhile`</a>
* Linq: <a href="http://msdn.microsoft.com/en-us/library/system.reactive.linq.observable.skipwhile(v=vs.103).aspx">`SkipWhile`</a>
* <a href="http://www.introtorx.com/Content/v1.0.10621.0/05_Filtering.html#SkipWhileTakeWhile">Introduction to Rx: SkipWhile and TakeWhile</a>

***

## take( )
#### emit only the first _n_ items emitted by an Observable
[[images/rx-operators/take.png]]

You can choose to pay attention only to the first _n_ items emitted by an Observable by calling its `take(n)` method. That method returns an Observable that will invoke an Observer’s `onNext` method a maximum of _n_ times before invoking `onCompleted`. For example,

```groovy
numbers = Observable.from([1, 2, 3, 4, 5, 6, 7, 8, 9]);

Observable.take(numbers, 3).subscribe(
  { println(it); },                  // onNext
  { println("Error encountered"); }, // onError
  { println("Sequence complete"); }  // onCompleted
);
```
```
1
2
3
Sequence complete
```

In addition to calling `take( )` as a stand-alone method, you can also call it as a method of an Observable, so, in the example above, instead of 

```groovy
Observable.take(numbers, 3) ...
```
you could instead write 
```groovy
numbers.take(3) ...
```

If you call `take(n)` on an Observable, and that Observable emits _fewer_ than _n_ items before completing, the new, `take`-modified Observable will _not_ throw an exception or invoke `onError()`, but will merely emit this same fewer number of items before it completes.

#### see also:
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#take(int)">`take(num)`</a>
* RxJS: <a href="https://github.com/Reactive-Extensions/RxJS/wiki/Observable#wiki-take">`take`</a>
* Linq: <a href="http://msdn.microsoft.com/en-us/library/hh229852(v=vs.103).aspx">`Take`</a>
* <a href="http://www.introtorx.com/Content/v1.0.10621.0/05_Filtering.html#SkipAndTake">Introduction to Rx: Skip and Take</a>

***

## takeWhile( ) and takeWhileWithIndex( )
#### emit items emitted by an Observable as long as a specified condition is true, then skip the remainder
[[images/rx-operators/takeWhile.png]]

The `takeWhile( )` method returns an Observable that mirrors the behavior of the source Observable until such time as a function applied to an item emitted by that Observable returns `false`, whereupon the new Observable invokes `onCompleted( )`.

```groovy
numbers = Observable.from( [1, 2, 3, 4, 5, 6, 7, 8, 9] );

numbers.takeWhile({ ((it < 6) || (0 == (it % 2))) }).subscribe(
  { println(it); },                  // onNext
  { println("Error encountered"); }, // onError
  { println("Sequence complete"); }  // onCompleted
);
```
```
1
2
3
4
5
6
Sequence complete
```

[[images/rx-operators/takeWhileWithIndex.png]]
The `takeWhileWithIndex( )` method is similar, but your function takes an additional parameter: the (zero-based) index of the item being emitted by the source Observable.
```groovy
numbers = Observable.from( [1, 2, 3, 4, 5, 6, 7, 8, 9] );

numbers.takeWhileWithIndex({ it, index -> ((it < 6) || (index < 5)) }).subscribe(
  { println(it); },                  // onNext
  { println("Error encountered"); }, // onError
  { println("Sequence complete"); }  // onCompleted
);
```
```
1
2
3
4
5
Sequence complete
```

#### see also:
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#takeWhile(rx.util.functions.Func1)">`takeWhile(predicate)`</a>
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#takeWhileWithIndex(rx.util.functions.Func2)">`takeWhileWithIndex(predicate)`</a>
* RxJS: <a href="https://github.com/Reactive-Extensions/RxJS/wiki/Observable#wiki-takeWhile">`takeWhile`</a>
* Linq: <a href="http://msdn.microsoft.com/en-us/library/system.reactive.linq.observable.takewhile(v=vs.103).aspx">`TakeWhile`</a>
* <a href="http://www.introtorx.com/Content/v1.0.10621.0/05_Filtering.html#SkipWhileTakeWhile">Introduction to Rx: SkipWhile and TakeWhile</a>

***

## first( )
#### emit only the first item emitted by an Observable, or the first item that meets some condition
[[images/rx-operators/first.png]]

To create an Observable that emits only the first item emitted by a source Observable (if any), use the `first( )` method.

[[images/rx-operators/firstN.png]]
You can also pass a function to this method that evaluates items as they are emitted by the source Observable, in which case `first( )` will create an Observable that emits the first such item for which your function returns `true` (if any).

#### see also:
* RxJS: <a href="https://github.com/Reactive-Extensions/RxJS/wiki/Observable#wiki-first">`first`</a>
* Linq: <a href="http://msdn.microsoft.com/en-us/library/system.reactive.linq.observable.first(v=vs.103).aspx">`First`</a>
* <a href="http://www.introtorx.com/Content/v1.0.10621.0/07_Aggregation.html#First">Introduction to Rx: First</a>

***

## firstOrDefault( )
#### emit only the first item emitted by an Observable, or the first item that meets some condition, or a default value if the source Observable is empty
[[images/rx-operators/firstOrDefault.png]]

To create an Observable that emits only the first item emitted by a source Observable (or a default value if the source Observable is empty), use the `firstOrDefault( )` method.

[[images/rx-operators/firstOrDefaultN.png]]
You can also pass a function to this method that evaluates items as they are emitted by the source Observable, in which case `firstOrDefault( )` will create an Observable that emits the first such item for which your function returns `true` (or the supplied default value if no such item is emitted).

#### see also:
* RxJS: <a href="https://github.com/Reactive-Extensions/RxJS/wiki/Observable#wiki-firstOrDefault">`firstOrDefault`</a>
* Linq: <a href="http://msdn.microsoft.com/en-us/library/system.reactive.linq.observable.firstordefault(v=vs.103).aspx">`FirstOrDefault`</a>
* <a href="http://www.introtorx.com/Content/v1.0.10621.0/07_Aggregation.html#First">Introduction to Rx: First</a>


***

## elementAt( )
#### emit item _n_ emitted by the source Observable
[[images/rx-operators/elementAt.png]]

Pass `elementAt( )` a zero-based index value and it will emit the solitary item from the source Observable's sequence that matches that index value (for example, if you pass the index value 5, `elementAt( )` will emit the sixth item emitted by the source Observable).  If you pass in a negative index value, or if the source Observable emits fewer than _index value_ + 1 items, `elementAt( )` will throw an <code>IndexOutOfBoundsException</code>.

#### see also:
* RxJS: <a href="https://github.com/Reactive-Extensions/RxJS/wiki/Observable#elementat">`elementAt`</a>
* Linq: <a href="http://msdn.microsoft.com/en-us/library/hh229725(v=vs.103).aspx">`ElementAt`</a>
* <a href="http://www.introtorx.com/Content/v1.0.10621.0/06_Inspection.html#ElementAt">Introduction to Rx: ElementAt</a>

***

## elementAtOrDefault( )
#### emit item _n_ emitted by the source Observable, or a default item if the source Observable emits fewer than _n_ items
[[images/rx-operators/elementAtOrDefault.png]]

Pass `elementAtOrDefault( )` a zero-based index value and it will emit the solitary item from the source Observable's sequence that matches that index value (for example, if you pass the index value 5, `elementAtOrDefault( )` will emit the sixth item emitted by the source Observable).  If you pass in a negative index value, `elementAtOrDefault( )` will throw an <code>IndexOutOfBoundsException</code>. If the source Observable emits fewer than _index value_ + 1 items, `elementAtOrDefault( )` will emit the default value you pass in (you must also pass in a type for this value that is appropriate to what type your Observers expect to observe).

#### see also:
* RxJS: <a href="https://github.com/Reactive-Extensions/RxJS/wiki/Observable#elementatordefault">`elementAtOrDefault`</a>
* Linq: <a href="http://msdn.microsoft.com/en-us/library/hh229845(v=vs.103).aspx">`ElementAtOrDefault`</a>

***

## sample( ) or throttleLast( )
#### emit the most recent items emitted by an Observable within periodic time intervals
[[images/rx-operators/sample.png]]

Use the `sample( )` method to periodically look at an Observable to see what item it is emitting at a particular time.

The following code constructs an Observable that emits the numbers between one and a million, and then samples that Observable every ten milliseconds to see what number it is emitting at that moment.
```groovy
def numbers = Observable.range( 1, 1000000 );
 
numbers.sample(10, java.util.concurrent.TimeUnit.MILLISECONDS).subscribe(
  { println(it); },                  // onNext
  { println("Error encountered"); }, // onError
  { println("Sequence complete"); }  // onCompleted
);
```
```
339707
547810
891282
Sequence complete
```

#### see also:
* RxJS: <a href="https://github.com/Reactive-Extensions/RxJS/wiki/Observable#wiki-sample">`sample`</a>
* Linq: <a href="http://msdn.microsoft.com/en-us/library/system.reactive.linq.observable.sample(v=vs.103).aspx">`Sample`</a>
* <a href="http://www.introtorx.com/Content/v1.0.10621.0/13_TimeShiftedSequences.html#Sample">Introduction to Rx: Sample</a>

***

## throttleFirst( )
#### emit the first items emitted by an Observable within periodic time intervals
[[images/rx-operators/throttleFirst.png]]

Use the `throttleFirst( )` method to periodically look at an Observable to see what item it emitted first during a particular time span. The following code shows how an Observable can be modified by `throttleFirst( )`:

```groovy
PublishSubject<Integer> o = PublishSubject.create();
o.throttleFirst(500, TimeUnit.MILLISECONDS, s).subscribe(
    { println(it); },                  // onNext
    { println("Error encountered"); }, // onError
    { println("Sequence complete"); }  // onCompleted
);
// send events with simulated time increments
s.advanceTimeTo(0, TimeUnit.MILLISECONDS);
o.onNext(1); // deliver
o.onNext(2); // skip
s.advanceTimeTo(501, TimeUnit.MILLISECONDS);
o.onNext(3); // deliver
s.advanceTimeTo(600, TimeUnit.MILLISECONDS);
o.onNext(4); // skip
s.advanceTimeTo(700, TimeUnit.MILLISECONDS);
o.onNext(5); // skip
o.onNext(6); // skip
s.advanceTimeTo(1001, TimeUnit.MILLISECONDS);
o.onNext(7); // deliver
s.advanceTimeTo(1501, TimeUnit.MILLISECONDS);
o.onCompleted();
```
```
1
3
7
Sequence complete
```

***

## throttleWithTimeout( ) or debounce( )
#### only emit an item from the source Observable after a particular timespan has passed without the Observable emitting any other items
[[images/rx-operators/throttleWithTimeout.png]]

Use the `throttleWithTimeout( )` method to select only those items emitted by a source Observable that are not quickly superceded by other items.

#### see also:
* Linq: <a href="http://msdn.microsoft.com/en-us/library/system.reactive.linq.observable.throttle(v=vs.103).aspx">`Throttle`</a>
* <a href="http://www.introtorx.com/Content/v1.0.10621.0/13_TimeShiftedSequences.html#Throttle">Introduction to Rx: Throttle</a>

***

## distinct( )
#### suppress duplicate items emitted by the source Observable
[[images/rx-operators/distinct.png]]

Use the `distinct( )` method to remove duplicate items from a source Observable and only emit single examples of those items.

[[images/rx-operators/distinct.key.png]]

You can also pass a function or a comparator into `distinct( )` that customizes how it distinguishes between distinct and non-distinct items.

#### see also:
* RxJS: <a href="https://github.com/Reactive-Extensions/RxJS/wiki/Observable#wiki-distinct">`distinct`</a>
* Linq: <a href="http://msdn.microsoft.com/en-us/library/system.reactive.linq.observable.distinct(v=vs.103).aspx">`Distinct`</a>
* <a href="http://www.introtorx.com/Content/v1.0.10621.0/05_Filtering.html#Distinct">Introduction to Rx: Distinct and DistinctUntilChanged</a>

***

## distinctUntilChanged( )
#### suppress duplicate consecutive items emitted by the source Observable
[[images/rx-operators/distinctUntilChanged.png]]

Use the `distinctUntilChanged( )` method to remove duplicate consecutive items from a source Observable and only emit single examples of such items.

[[images/rx-operators/distinctUntilChanged.key.png]]

You can also pass a function or a comparator into `distinctUntilChanged( )` that customizes how it distinguishes between distinct and non-distinct items.

#### see also:
* RxJS: <a href="https://github.com/Reactive-Extensions/RxJS/wiki/Observable#wiki-distinctUntilChanged">`distinctUntilChanged`</a>
* Linq: <a href="http://msdn.microsoft.com/en-us/library/system.reactive.linq.observable.distinctuntilchanged(v=vs.103).aspx">`DistinctUntilChanged`</a>
* <a href="http://www.introtorx.com/Content/v1.0.10621.0/05_Filtering.html#Distinct">Introduction to Rx: Distinct and DistinctUntilChanged</a>

***

## ofClass( )
#### emit only those items from the source Observable that are of a particular class
[[images/rx-operators/ofClass.png]]

#### see also:
* Linq: <a href="http://msdn.microsoft.com/en-us/library/hh229380(v=vs.103).aspx">`OfType`</a>
* <a href="http://www.introtorx.com/Content/v1.0.10621.0/08_Transformation.html#CastAndOfType">Introduction to Rx: Cast and OfType</a>