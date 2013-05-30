This section explains operators you can use to filter and select elements from Observables.

* [**`filter( )` or `where( )`**](Filtering-Observables#filter-or-where) — filter elements emitted by an Observable
* [**`takeLast( )`**](Filtering-Observables#takelast) — only emit the last _n_ elements emitted by an Observable
* [**`skip( )`**](Filtering-Observables#skip) — ignore the first _n_ elements emitted by an Observable
* [**`take( )`**](Filtering-Observables#take) — emit only the first _n_ elements emitted by an Observable
* [**`sample( )`**](Filtering-Observables#sample) — emit items emitted by an Observable at a particular time interval
* [**`takeWhile( )` and `takeWhileWithIndex( )`**](Filtering-Observables#takewhile-and-takewhilewithindex) — emit items emitted an Observable as long as a specified condition is true, then skip the remainder

## filter( ) or where( )
#### filter elements from an Observable sequence
[[images/rx-operators/filter.png]]

You can filter an Observable, discarding any values that do not meet some test, by passing a filtering closure into the `filter( )` method. For example, the following code filters a list of integers, emitting only those that are even (that is, where the remainder from dividing the number by two is zero):

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

In addition to calling `filter( )` as a stand-alone method, you can also call it as a method of an Observable object, so, in the example above, instead of 

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
#### only emit the last _n_ elements emitted by an Observable
[[images/rx-operators/last.png]]

To convert an Observable that emits several objects into one that only emits the last _n_ of these objects before completing, use the `takeLast( )` method. For instance, in the following code, `takeLast( )` emits only the last integer in the list of integers represented by `numbers`:

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

In addition to calling `takeLast( )` as a stand-alone method, you can also call it as a method of an Observable object, so, in the example above, instead of 

```groovy
Observable.takeLast(numbers,1) ...
```
you could instead write
```groovy
numbers.takeLast(1) ...
```

## skip()
#### ignore the first _n_ elements emitted by an Observable
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

In addition to calling `skip( )` as a stand-alone method, you can also call it as a method of an Observable object, so, in the example above, instead of 

```groovy
Observable.skip(numbers, 3) ...
```
you could instead write 
```groovy
numbers.skip(3) ...
```

## take( )
#### emit only the first _n_ elements from an Observable sequence
[[images/rx-operators/take.png]]

You can choose to pay attention only to the first _n_ values emitted by an Observable by calling its `take(n)` method. That method returns an Observable that will call a subscribing observer’s `onNext` closure a maximum of _n_ times before calling `onCompleted`. For example,

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

In addition to calling `take( )` as a stand-alone method, you can also call it as a method of an Observable object, so, in the example above, instead of 

```groovy
Observable.take(numbers, 3) ...
```
you could instead write 
```groovy
numbers.take(3) ...
```

If you call `take(n)` on an Observable, and that Observable emits _fewer_ than _n_ items before completing, the new, `take`-modified Observable will _not_ throw an exception or invoke `onError()`, but will merely emit this same fewer number of items before it completes.

## sample( )
#### emit items emitted by an Observable at a particular time interval
[[images/rx-operators/sample.png]]

Use the `sample( )` method to periodically look at an Observable to see what object it is emitting at a particular time.

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

## takeWhile( ) and takeWhileWithIndex( )
#### emit items emitted an Observable as long as a specified condition is true, then skip the remainder
[[images/rx-operators/takeWhile.png]]

The `takeWhile( )` method returns an Observable that mirrors the behavior of the source Observable until such time as a closure applied to an object emitted by that observable returns `false`, whereupon the new Observable calls `onCompleted( )`.

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

The `takeWhileWithIndex( )` method is similar, but your closure takes an additional parameter: the (zero-based) index of the object being emitted by the source Observable.
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