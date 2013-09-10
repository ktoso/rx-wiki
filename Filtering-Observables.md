This section explains operators you can use to filter and select items emitted by Observables.

* [**`filter( )` or `where( )`**](Filtering-Observables#filter-or-where) — filter items emitted by an Observable
* [**`takeLast( )`**](Filtering-Observables#takelast) — only emit the last _n_ items emitted by an Observable
* [**`skip( )`**](Filtering-Observables#skip) — ignore the first _n_ items emitted by an Observable
* [**`take( )`**](Filtering-Observables#take) — emit only the first _n_ items emitted by an Observable
* [**`sample( )`**](Filtering-Observables#sample) — emit the most recent items emitted by an Observable within periodic time intervals
* [**`throttleFirst( )`**](Filtering-Observables#throttlefirst) — emit the first items emitted by an Observable within periodic time intervals
* [**`throttleWithTimeout( )`**](Filtering-Observables#throttlewithtimeout) — only emit an item from the source Observable after a particular timespan has passed without the Observable emitting any other items
* [**`takeWhile( )` and `takeWhileWithIndex( )`**](Filtering-Observables#takewhile-and-takewhilewithindex) — emit items emitted by an Observable as long as a specified condition is true, then skip the remainder
* [**`skipWhile( )` and `skipWhileWithIndex( )`**](Filtering-Observables#skipwhile-and-skipwhilewithindex) — discard items emitted by an Observable until a specified condition is false, then emit the remainder
* [**`first( )`**](Filtering-Observables#first) — emit only the first item emitted by an Observable, or the first item that meets some condition
* [**`firstOrDefault( )`**](Filtering-Observables#firstordefault) — emit only the first item emitted by an Observable, or the first item that meets some condition, or a default value if the source Observable is empty

## filter( ) or where( )
#### filter items emitted by an Observable
[[images/rx-operators/filter.png]]

You can filter an Observable, discarding any items that do not meet some test, by passing a filtering function into the `filter( )` method. For example, the following code filters a list of integers, emitting only those that are even (that is, where the remainder from dividing the number by two is zero):

```groovy
numbers = Observable.toObservable([1, 2, 3, 4, 5, 6, 7, 8, 9]);

Observable.filter(numbers, { 0 == (it % 2) }).subscribe(
  [ onNext:{ myWriter.println(it); },
    onCompleted:{ myWriter.println("Sequence complete"); },
    onError:{ myWriter.println("Error encountered"); } ]
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

numbers = Observable.toObservable([1, 2, 3, 4, 5, 6, 7, 8, 9]);

numbers.where(myisEven).subscribe(
  [ onNext:{ myWriter.println(it); },
    onCompleted:{ myWriter.println("Sequence complete"); },
    onError:{ myWriter.println("Error encountered"); } ]
);
```

## takeLast( )
#### only emit the last _n_ items emitted by an Observable
[[images/rx-operators/last.png]]

To convert an Observable that emits several items into one that only emits the last _n_ of these itemsbefore completing, use the `takeLast( )` method. For instance, in the following code, `takeLast( )` emits only the last integer in the list of integers represented by `numbers`:

```groovy
numbers = Observable.toObservable([1, 2, 3, 4, 5, 6, 7, 8, 9]);

Observable.takeLast(numbers,1).subscribe(
  [ onNext:{ myWriter.println(it); },
    onCompleted:{ myWriter.println("Sequence complete"); },
    onError:{ myWriter.println("Error encountered"); } ]
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

## skip()
#### ignore the first _n_ items emitted by an Observable
[[images/rx-operators/skip.png]]

You can ignore the first _n_ items emitted by an Observable and attend only to those items that come after, by modifying the Observable with the `skip(n)` method.

```groovy
numbers = Observable.toObservable([1, 2, 3, 4, 5, 6, 7, 8, 9]);

Observable.skip(numbers, 3).subscribe(
  [ onNext:{ myWriter.println(it); },
    onCompleted:{ myWriter.println("Sequence complete"); },
    onError:{ myWriter.println("Error encountered"); } ]
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

## take( )
#### emit only the first _n_ items emitted by an Observable
[[images/rx-operators/take.png]]

You can choose to pay attention only to the first _n_ items emitted by an Observable by calling its `take(n)` method. That method returns an Observable that will invoke an Observer’s `onNext` method a maximum of _n_ times before invoking `onCompleted`. For example,

```groovy
numbers = Observable.toObservable([1, 2, 3, 4, 5, 6, 7, 8, 9]);

Observable.take(numbers, 3).subscribe(
  [ onNext:{ myWriter.println(it); },
    onCompleted:{ myWriter.println("Sequence complete"); },
    onError:{ myWriter.println("Error encountered"); } ]
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

## sample( )
#### emit the most recent items emitted by an Observable within periodic time intervals
[[images/rx-operators/sample.png]]

Use the `sample( )` method to periodically look at an Observable to see what item it is emitting at a particular time.

The following code constructs an Observable that emits the numbers between one and a million, and then samples that Observable every ten milliseconds to see what number it is emitting at that moment.
```groovy
def numbers = Observable.range( 1, 1000000 );
 
numbers.sample(10, java.util.concurrent.TimeUnit.MILLISECONDS).subscribe(
  [ onNext:{ myWriter.println(it); },
    onCompleted:{ myWriter.println("Sequence complete"); },
    onError:{ myWriter.println("Error encountered"); } ]
);
```
```
339707
547810
891282
Sequence complete
```

## throttleFirst( )
#### emit the first items emitted by an Observable within periodic time intervals
[[images/rx-operators/throttleFirst.png]]

Use the `throttleFirst( )` method to periodically look at an Observable to see what item it emitted first during a particular time span. The following code shows how an Observable can be modified by `throttleFirst( )`:

```groovy
    PublishSubject<Integer> o = PublishSubject.create();
    o.throttleFirst(500, TimeUnit.MILLISECONDS, s).subscribe(
        [ onNext:{ myWriter.println(it); },
          onCompleted:{ myWriter.println("Sequence complete"); },
          onError:{ myWriter.println("Error encountered"); } ]
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

## throttleWithTimeout( )
#### only emit an item from the source Observable after a particular timespan has passed without the Observable emitting any other items
[[images/rx-operators/throttleWithTimeout.png]]

Use the `throttleWithTimeout( )` method to select only those items emitted by a source Observable that are not quickly superceded by other items.

## takeWhile( ) and takeWhileWithIndex( )
#### emit items emitted by an Observable as long as a specified condition is true, then skip the remainder
[[images/rx-operators/takeWhile.png]]

The `takeWhile( )` method returns an Observable that mirrors the behavior of the source Observable until such time as a function applied to an item emitted by that Observable returns `false`, whereupon the new Observable invokes `onCompleted( )`.

```groovy
numbers = Observable.toObservable( [1, 2, 3, 4, 5, 6, 7, 8, 9] );

numbers.takeWhile({ ((it < 6) || (0 == (it % 2))) }).subscribe(
  [ onNext:{ myWriter.println(it); },
    onCompleted:{ myWriter.println("Sequence complete"); },
    onError:{ myWriter.println("Error encountered"); } ]
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

The `takeWhileWithIndex( )` method is similar, but your function takes an additional parameter: the (zero-based) index of the item being emitted by the source Observable.
```groovy
numbers = Observable.toObservable( [1, 2, 3, 4, 5, 6, 7, 8, 9] );

numbers.takeWhileWithIndex({ it, index -> ((it < 6) || (index < 5)) }).subscribe(
  [ onNext:{ myWriter.println(it); },
    onCompleted:{ myWriter.println("Sequence complete"); },
    onError:{ myWriter.println("Error encountered"); } ]
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

## skipWhile( ) and skipWhileWithIndex( )
#### discard items emitted by an Observable until a specified condition is false, then emit the remainder
[[images/rx-operators/skipWhile.png]]

The `skipWhile( )` method returns an Observable that discards items emitted by the source Observable until such time as a function applied to an item emitted by that Observable returns `false`, whereupon the new Observable emits that item and the remainder of the items emitted by the source Observable.

```groovy
numbers = Observable.toObservable( [1, 2, 3, 4, 5, 6, 7, 8, 9] );

numbers.skipWhile({ (0 == (it % 5)) }).subscribe(
  [ onNext:{ myWriter.println(it); },
    onCompleted:{ myWriter.println("Sequence complete"); },
    onError:{ myWriter.println("Error encountered"); } ]
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
numbers = Observable.toObservable( [1, 2, 3, 4, 5, 6, 7, 8, 9] );

numbers.skipWhileWithIndex({ it, index -> ((it < 6) || (index < 5)) }).subscribe(
  [ onNext:{ myWriter.println(it); },
    onCompleted:{ myWriter.println("Sequence complete"); },
    onError:{ myWriter.println("Error encountered"); } ]
);
```
```
6
7
8
9
Sequence complete
```

## first( )
#### emit only the first item emitted by an Observable, or the first item that meets some condition
[[images/rx-operators/first.png]]

To create an Observable that emits only the first item emitted by a source Observable (if any), use the `first( )` method.

[[images/rx-operators/firstN.png]]
You can also pass a function to this method that evaluates items as they are emitted by the source Observable, in which case `first( )` will create an Observable that emits the first such item for which your function returns `true` (if any).

## firstOrDefault( )
#### emit only the first item emitted by an Observable, or the first item that meets some condition, or a default value if the source Observable is empty
[[images/rx-operators/firstOrDefault.png]]

To create an Observable that emits only the first item emitted by a source Observable (or a default value if the source Observable is empty), use the `firstOrDefault( )` method.

[[images/rx-operators/firstOrDefaultN.png]]
You can also pass a function to this method that evaluates items as they are emitted by the source Observable, in which case `firstOrDefault( )` will create an Observable that emits the first such item for which your function returns `true` (or the supplied default value if no such item is emitted).