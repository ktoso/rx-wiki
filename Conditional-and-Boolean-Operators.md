This section explains operators with which you conditionally emit or transform Observables, or can do boolean evaluations of them:

### Conditional Operators
* [**`amb( )`**](Conditional-and-Boolean-Operators#amb) — given two or more source Observables, emits all of the items from the first of these Observables to emit an item
* [**`defaultIfEmpty( )`**](Conditional-and-Boolean-Operators#defaultifempty) — emit items from the source Observable, or emit a default item if the source Observable completes after emitting no items
* [**`skipWhile( )` and `skipWhileWithIndex( )`**](Conditional-and-Boolean-Operators#skipwhile-and-skipwhilewithindex) — discard items emitted by an Observable until a specified condition is false, then emit the remainder
* [**`takeUntil( )`**](Conditional-and-Boolean-Operators#takeuntil) — emits the items from the source Observable until a second Observable emits an item
* [**`takeWhile( )` and `takeWhileWithIndex( )`**](Conditional-and-Boolean-Operators#takewhile-and-takewhilewithindex) — emit items emitted by an Observable as long as a specified condition is true, then skip the remainder

### Boolean Operators

***

## amb( )
#### given two or more source Observables, emits all of the items from the first of these Observables to emit an item
[[images/rx-operators/amb.png]]

When you pass a number of source Observables to `amb( )`, it will pass through the emissions and messages of exactly one of these Observables: the first one that emits an item to `amb( )`. It will ignore and discard the emissions of all of the other source Observables.

#### see also:
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#amb(java.lang.Iterable)">`amb()`</a> (several varieties)
* RxJS: <a href="https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md#rxobservableambargs">`amb`</a>
* Linq: <a href="http://msdn.microsoft.com/en-us/library/system.reactive.linq.observable.amb(v=vs.103).aspx">`Amb`</a>

***

## defaultIfEmpty( )
#### emit items from the source Observable, or emit a default item if the source Observable completes after emitting no items
[[images/rx-operators/defaultIfEmpty.png]]

#### see also:
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#defaultIfEmpty(T)">`defaultIfEmpty(default)`</a>
* RxJS: <a href="https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md#rxobservableprototypedefaultifemptydefaultvalue">`defaultIfEmpty`</a>
* Linq: <a href="http://msdn.microsoft.com/en-us/library/system.reactive.linq.observable.defaultifempty.aspx">`DefaultIfEmpty`</a>
* <a href="http://www.introtorx.com/Content/v1.0.10621.0/06_Inspection.html#DefaultIfEmpty">Introduction to Rx: DefaultIfEmpty</a>

***

## skipWhile( ) and skipWhileWithIndex( )
#### discard items emitted by an Observable until a specified condition is false, then emit the remainder
[[images/rx-operators/skipWhile.png]]

The `skipWhile( )` method returns an Observable that discards items emitted by the source Observable until such time as a function applied to an item emitted by that Observable returns `false`, whereupon the new Observable emits that item and the remainder of the items emitted by the source Observable.

```groovy
numbers = Observable.from( [1, 2, 3, 4, 5, 6, 7, 8, 9] );

numbers.skipWhile({ (5 != it) }).subscribe(
  { println(it); },                          // onNext
  { println("Error: " + it.getMessage()); }, // onError
  { println("Sequence complete"); }          // onCompleted
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
  { println(it); },                          // onNext
  { println("Error: " + it.getMessage()); }, // onError
  { println("Sequence complete"); }          // onCompleted
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
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#skipWhile(rx.util.functions.Func1)">`skipWhile(predicate)`</a>
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#skipWhileWithIndex(rx.util.functions.Func2)">`skipWhileWithIndex(predicate)`</a>
* Linq: <a href="http://msdn.microsoft.com/en-us/library/system.reactive.linq.observable.skipwhile.aspx">`SkipWhile`</a>
* RxJS: <a href="https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md#rxobservableprototypeskipwhilepredicate-thisarg">`skipWhile`</a>
* <a href="http://www.introtorx.com/Content/v1.0.10621.0/05_Filtering.html#SkipWhileTakeWhile">Introduction to Rx: SkipWhile and TakeWhile</a>

***

## takeUntil( )
#### emits the items from the source Observable until another Observable emits an item
[[images/rx-operators/takeUntil.png]]

#### see also:
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#takeUntil(rx.Observable)">`takeUntil(other)`</a>
* RxJS: <a href="https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md#rxobservableprototypetakeuntilother">`takeUntil`</a>
* Linq: <a href="http://msdn.microsoft.com/en-us/library/hh229530.aspx">`TakeUntil`</a>

***

## takeWhile( ) and takeWhileWithIndex( )
#### emit items emitted by an Observable as long as a specified condition is true, then skip the remainder
[[images/rx-operators/takeWhile.png]]

The `takeWhile( )` method returns an Observable that mirrors the behavior of the source Observable until such time as a function applied to an item emitted by that Observable returns `false`, whereupon the new Observable invokes `onCompleted( )`.

```groovy
numbers = Observable.from( [1, 2, 3, 4, 5, 6, 7, 8, 9] );

numbers.takeWhile({ ((it < 6) || (0 == (it % 2))) }).subscribe(
  { println(it); },                          // onNext
  { println("Error: " + it.getMessage()); }, // onError
  { println("Sequence complete"); }          // onCompleted
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
  { println(it); },                          // onNext
  { println("Error: " + it.getMessage()); }, // onError
  { println("Sequence complete"); }          // onCompleted
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
* Linq: <a href="http://msdn.microsoft.com/en-us/library/system.reactive.linq.observable.takewhile.aspx">`TakeWhile`</a>
* RxJS: <a href="https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md#rxobservableprototypetakewhilepredicate-thisarg">`takeWhile`</a>
* <a href="http://www.introtorx.com/Content/v1.0.10621.0/05_Filtering.html#SkipWhileTakeWhile">Introduction to Rx: SkipWhile and TakeWhile</a>

