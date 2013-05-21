This section explains operators you can use to filter and select elements from Observables.

* **`filter()` or `where()`** — filter elements emitted by an Observable
* **`takeLast()`** — only emit the last _n_ elements emitted by an Observable
* **`skip()`** — ignore the first _n_ elements emitted by an Observable
* **`take()`** — emit only the first _n_ elements emitted by an Observable
* **`sample()`** — emit items emitted by an Observable at a particular time interval
* **`takeWhile()`** — emit items emitted an Observable as long as a specified condition is true
* **`takeWhileWithIndex()`** — emit items emitted an Observable as long as a specified condition is true, then skip the remainder

## filter() or where()
#### filter elements from an Observable sequence

[[images/rx-operators/filter.png]]

You can filter a Observable, discarding any values that do not meet some test, by passing a filtering closure into the `filter()` method. For example, the following code filters a list of integers, emitting only those that are even (that is, where the remainder from dividing the number by two is zero):

```groovy
numbers = Observable.toObservable([1, 2, 3, 4, 5, 6, 7, 8, 9]);

Observable.filter(numbers, { 0 == (it % 2) }).subscribe(
  [ onNext:{ response.getWriter().println(it); },
    onCompleted:{ response.getWriter().println("Sequence complete"); },
    onError:{ response.getWriter().println("Error encountered"); } ]
);

2
4
6
8
Sequence complete
```

In addition to calling `filter()` as a stand-alone method, you can also call it as a method of a Observable object, so, in the example above, instead of 

```groovy
Observable.filter(numbers, { 0 == (it %2) }) ...
```
you could instead write 

```groovy
numbers.filter({ 0 == (it % 2) }) ...
```

## takeLast()
#### only emit the last _n_ elements emitted by an Observable

[[images/rx-operators/last.png]]

To convert a Observable that emits several objects into one that only emits the last _n_ of these objects before completing, use the `takeLast()` method. For instance, in the following code, `takeLast()` emits only the last integer in the list of integers represented by `numbers`:

```groovy
numbers = Observable.toObservable([1, 2, 3, 4, 5, 6, 7, 8, 9]);

Observable.takeLast(numbers,1).subscribe(
  [ onNext:{ response.getWriter().println(it); },
    onCompleted:{ response.getWriter().println("Sequence complete"); },
    onError:{ response.getWriter().println("Error encountered"); } ]
);

9
Sequence complete
```

In addition to calling `takeLast()` as a stand-alone method, you can also call it as a method of a Observable object, so, in the example above, instead of 

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

You can ignore the first _n_ items emitted by a Observable and attend only to those items that come after, by modifying the Observable with the `Observable.skip(n)` method.

```groovy
numbers = Observable.toObservable([1, 2, 3, 4, 5, 6, 7, 8, 9]);

Observable.skip(numbers, 3).subscribe(
  [ onNext:{ response.getWriter().println(it); },
    onCompleted:{ response.getWriter().println("Sequence complete"); },
    onError:{ response.getWriter().println("Error encountered"); } ]
);

4
5
6
7
8
9
Sequence complete
```

In addition to calling `skip()` as a stand-alone method, you can also call it as a method of a Observable object, so, in the example above, instead of 

```groovy
Observable.skip(numbers, 3) ...
```

you could instead write 

```groovy
numbers.skip(3) ...
```

## take()
#### emit only the first _n_ elements from an Observable sequence

[[images/rx-operators/take.png]]

You can choose to pay attention only to the first _n_ values emitted by a Observable by calling its `take(n)` method. That method returns a Observable that will call a subscribing observer’s `onNext` closure a maximum of _n_ times before calling `onCompleted`. For example,

```groovy
numbers = Observable.toObservable([1, 2, 3, 4, 5, 6, 7, 8, 9]);

Observable.take(numbers, 3).subscribe(
  [ onNext:{ response.getWriter().println(it); },
    onCompleted:{ response.getWriter().println("Sequence complete"); },
    onError:{ response.getWriter().println("Error encountered"); } ]
);

1
2
3
Sequence complete
```

In addition to calling `take()` as a stand-alone method, you can also call it as a method of a Observable object, so, in the example above, instead of 

```groovy
Observable.take(numbers, 3) ...
```
you could instead write 

```groovy
numbers.take(3) ...
```

If you call `take(n)` on a Observable, and that Observable emits _fewer_ than _n_ items before completing, the new, `take`-modified Observable will _not_ throw an error, but will merely emit this same fewer number of items before it completes.

## sample()
#### emit items emitted by an Observable at a particular time interval

## takeWhile()
#### emit items emitted an Observable as long as a specified condition is true
The `takeWhile()` method returns an Observable that mirrors the behavior of the source Observable until such time as a closure applied to an object emitted by that observable returns `false`, whereupon the new Observable calls `onCompleted()`.

```groovy
numbers = Observable.toObservable( [1, 2, 3, 4, 5, 6, 7, 8, 9] );

numbers.takeWhile({ ((it < 6) || (0 == (it % 2))) }).subscribe(
  [onNext:{ response.getWriter().println( it ); },
   onCompleted:{ response.getWriter().println( "Sequence complete" ); },
   onError:{ response.getWriter().println( "Error encountered" ); } ]
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

## takeWhileWithIndex()
#### emit items emitted an Observable as long as a specified condition is true, then skip the remainder