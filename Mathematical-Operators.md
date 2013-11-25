This section explains operators that perform mathematical operations on the items emitted by Observables.

* [**`count( )`**](Mathematical-Operators#count) — counts the number of items emitted by an Observable and emits this count
* [**`sum( )`**](Mathematical-Operators#sum) — adds the Integers emitted by an Observable and emits this sum
* [**`sumLongs( )`**](Mathematical-Operators#sum) — adds the Longs emitted by an Observable and emits this sum
* [**`sumFloats( )`**](Mathematical-Operators#sum) — adds the Floats emitted by an Observable and emits this sum
* [**`sumDoubles( )`**](Mathematical-Operators#sum) — adds the Floats emitted by an Observable and emits this sum
* [**`average( )`**](Mathematical-Operators#average) — calculates the average of Integers emitted by an Observable and emits this average
* [**`averageLongs( )`**](Mathematical-Operators#average) — calculates the average of Longs emitted by an Observable and emits this average
* [**`averageFloats( )`**](Mathematical-Operators#average) — calculates the average of Floats emitted by an Observable and emits this average
* [**`averageDoubles( )`**](Mathematical-Operators#average) — calculates the average of Doubles emitted by an Observable and emits this average
* [**`min( )`**](Mathematical-Operators#min) — emits the minimum value emitted by a source Observable
* [**`minBy( )`**](Mathematical-Operators#minby) — emits the item emitted by the source Observable that has the minimum key value
* [**`max( )`**](Mathematical-Operators#max) — emits the maximum value emitted by a source Observable
* [**`maxBy( )`**](Mathematical-Operators#maxby) — emits the item emitted by the source Observable that has the maximum key value

***

## count( )
#### counts the number of items emitted by an Observable and emits this count
[[images/rx-operators/count.png]]

The `count( )` method returns an Observable that emits a single item: an Integer that represents the total number of items emitted by the source Observable, as shown in the following sample code:
```groovy
def myObservable = Observable.create({ anObserver ->
  anObserver.onNext('Three');
  anObserver.onNext('Two');
  anObserver.onNext('One');
  anObserver.onCompleted();
});

myObservable.count().subscribe(
  { println(it); },                  // onNext
  { println("Error encountered"); }, // onError
  { println("Sequence complete"); }  // onCompleted
);
```
```
3
Sequence complete
```

#### see also:
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#count()">`count()`</a>
* RxJS: <a href="https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md#rxobservableprototypecountpredicate">`count`</a>
* Linq: <a href="http://msdn.microsoft.com/en-us/library/hh229470.aspx">`Count`</a>
* <a href="http://www.introtorx.com/Content/v1.0.10621.0/07_Aggregation.html#Count">Introduction to Rx: Count</a>

***

## sum( )
#### adds the numbers emitted by an Observable and emits this sum
[[images/rx-operators/sum.png]]

The `sum( )` method returns an Observable that adds the Integers emitted by a source Observable and then emits this sum as an Integer, as shown in the following sample code:
```groovy
def myObservable = Observable.create({ anObserver ->
  anObserver.onNext(4);
  anObserver.onNext(3);
  anObserver.onNext(2);
  anObserver.onNext(1);
  anObserver.onCompleted();
});

myObservable.sum().subscribe(
  { println(it); },                  // onNext
  { println("Error encountered"); }, // onError
  { println("Sequence complete"); }  // onCompleted
);
```
```
10
Sequence complete
```
There are also specialized "sum" methods for Longs, Floats, and Doubles (`sumLongs( )`, `sumFloats( )`, and `sumDoubles( )`).

#### see also:
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#sum(rx.Observable)">sum()</a>, <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#sumLongs(rx.Observable)">sumLongs()</a>, <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#sumFloats(rx.Observable)">sumFloats()</a>, and <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#sumDoubles(rx.Observable)">sumDoubles()</a>
* RxJS: <a href="https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md#rxobservableprototypesumkeyselector-thisarg">`sum`</a>
* Linq: <a href="http://msdn.microsoft.com/en-us/library/system.reactive.linq.observable.sum.aspx">`Sum`</a>
* <a href="http://www.introtorx.com/Content/v1.0.10621.0/07_Aggregation.html#MaxAndMin">Introduction to Rx: Min, Max, Sum, and Average</a>

***

## average( )
#### calculates the average of numbers emitted by an Observable and emits this average
[[images/rx-operators/average.png]]

The `average( )` method returns an Observable that calculates the average of the Integers emitted by a source Observable and then emits this average as an Integer, as shown in the following sample code:
```groovy
def myObservable = Observable.create({ anObserver ->
  anObserver.onNext(4);
  anObserver.onNext(3);
  anObserver.onNext(2);
  anObserver.onNext(1);
  anObserver.onCompleted();
});

myObservable.average().subscribe(
  { println(it); },                  // onNext
  { println("Error encountered"); }, // onError
  { println("Sequence complete"); }  // onCompleted
);
```
```
2
Sequence complete
```
There are also specialized "average" methods for Longs, Floats, and Doubles (`averageLongs( )`, `averageFloats( )`, and `averageDoubles( )`).

Note that these methods will fail with an `IllegalArgumentException` if the source Observable does not emit any items.

#### see also:
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#average(rx.Observable)">`average()`</a>, <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#averageLongs(rx.Observable)">`averageLongs()`</a>, <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#averageFloats(rx.Observable)">`averageFloats()`</a>, and <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#averageDoubles(rx.Observable)">`averageDoubles()`</a>
* RxJS: <a href="https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md#rxobservableprototypeaverageselector">`average`</a>
* Linq: <a href="http://msdn.microsoft.com/en-us/library/system.reactive.linq.observable.average.aspx">`Average`</a>
* <a href="http://www.introtorx.com/Content/v1.0.10621.0/07_Aggregation.html#MaxAndMin">Introduction to Rx: Min, Max, Sum, and Average</a>

***

## min( )
#### emits the minimum value emitted by a source Observable
[[images/rx-operators/min.png]]

The `min( )` operator waits until the source Observable completes, and then emits the item emitted by the source Observable that had the lowest value, before itself completing.  If more than one item has this minimum value, `min( )` emits the last such item. You may optionally pass in a comparator that `min( )` will use to determine the minimum of two emitted items.

#### see also: 
* Linq: <a href="http://msdn.microsoft.com/en-us/library/system.reactive.linq.observable.min.aspx">`Min`</a>
* RxJS: <a href="https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md#rxobservableprototypemincomparer">`min`</a>
* <a href="http://www.introtorx.com/Content/v1.0.10621.0/07_Aggregation.html#MaxAndMin">Intro to Rx: Max and Min</a>

***

##  minBy( )
#### emits the item emitted by the source Observable that has the minimum key value
[[images/rx-operators/minBy.png]]

The `minBy( )` operator is similar to `min( )` but instead of emitting the minimum item emitted by the source Observable, it emits the last item from the source Observable that has the minimum key, where that key is generated by a function applied to each item. You supply this function.

#### see also: 
* Linq: <a href="http://msdn.microsoft.com/en-us/library/system.reactive.linq.observable.minby.aspx">`MinBy`</a>
* RxJS: <a href="https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md#rxobservableprototypeminbykeyselector-comparer">`minBy`</a>
* <a href="http://www.introtorx.com/Content/v1.0.10621.0/07_Aggregation.html#MinByMaxBy">Intro to Rx: MinBy and MaxBy</a>

***

## max( )
#### emits the maximum value emitted by a source Observable
[[images/rx-operators/max.png]]

The `max( )` operator waits until the source Observable completes, and then emits the item emitted by the source Observable that had the highest value, before itself completing. If more than one item has this maximum value, `max( )` emits the last such item. You may optionally pass in a comparator that `max( )` will use to determine the maximum of two emitted items.

#### see also: 
* Linq: <a href="http://msdn.microsoft.com/en-us/library/system.reactive.linq.observable.max.aspx">`Max`</a>
* RxJS: <a href="https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md#rxobservableprototypemaxcomparer">`max`</a>
* <a href="http://www.introtorx.com/Content/v1.0.10621.0/07_Aggregation.html#MaxAndMin">Intro to Rx: Max and Min</a>

***

## maxBy( )
#### emits the item emitted by the source Observable that has the maximum key value
[[images/rx-operators/maxBy.png]]

The `maxBy( )` operator is similar to `max( )` but instead of emitting the maximum item emitted by the source Observable, it emits the last item from the source Observable that has the maximum key, where that key is generated by a function applied to each item. You supply this function.

#### see also:
* Linq: <a href="http://msdn.microsoft.com/en-us/library/system.reactive.linq.observable.maxby.aspx">`MaxBy`</a>
* RxJS: <a href="https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md#rxobservableprototypemaxbykeyselector-comparer">`maxBy`</a>
* <a href="http://www.introtorx.com/Content/v1.0.10621.0/07_Aggregation.html#MinByMaxBy">Intro to Rx: MinBy and MaxBy</a>
