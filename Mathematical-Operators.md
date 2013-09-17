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