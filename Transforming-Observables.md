This section explains Rx operators you can use to transform elements emitted by an Observable sequence.

* **`map()`** — transform the elements emitted by an Observable by applying a closure to each of them
* **`mapMany()` or `flatMap()`** — transform the elements emitted by an Observable into Observables, then flatten into an Observable sequence
* **`mapManyDelayError()`** — transform the elements emitted by an Observable into Observables, then flatten into an Observable sequence, waiting to report errors until all error-free observables have a chance to complete
* **`reduce()` or `aggregate()`** — apply a closure to each emitted element, sequentially, and emit only the final accumulated value
* **`scan()`** — apply a closure to each element of a sequence, sequentially, and emit each successive value
* **`groupBy()`** — divide an Observable into a set of Observables that emit groups of values from the original Observable, organized by key

## map()
#### transform the elements emitted by an Observable by applying a closure to each of them

[[images/rx-operators/map.png]]

The `map()` method applies a closure of your choosing to every object emitted by a Observable, and returns this transformation as a new Observable sequence. For example, the following code maps a closure that squares the incoming value onto the values in `numbers`:

```groovy
numbers = Observable.toObservable([1, 2, 3, 4, 5]);

Observable.map(numbers, {it * it}).subscribe(
  [ onNext:{ response.getWriter().println(it); },
    onCompleted:{ response.getWriter().println("Sequence complete"); },
    onError:{ response.getWriter().println("Error encountered"); } ]
);

1
4
9
16
25
Sequence complete
```

In addition to calling `map()` as a stand-alone method, you can also call it as a method of a Observable object, so, in the example above, instead of 

```groovy
Observable.map(numbers, { it * it }) ...
```

you could instead write 

```groovy
numbers.map({ it * it }) ...
```

## mapMany() or flatMap() & mapManyDelayError()
#### Transform the elements emitted by an Observable into Observables, then flatten into an Observable sequence

[[images/rx-operators/mapMany.png]]

The `mapMany()` method creates a new Observable sequence by applying a closure that you supply to each object in the original Observable sequence, where that closure is itself a Observable that emits elements, and then merges the results of that closure applied to every item emitted by the original Observable, emitting these merged results as its own sequence.

This method is useful, for example, when you have a Observable that emits a series of objects that themselves have Observable members or are in other ways transformable into Observables, so that you can create a new Observable that emits the complete collection of items emitted by the sub-Observables of these objects.

```groovy
// this closure is a Observable that emits three numbers
numbers   = Observable.toObservable([1, 2, 3]);
// this closure is a Observable that emits three numbers based on what number it is passed
multiples = { n -> Observable.toObservable([ n*1, n*2, n*3 ]) };   

numbers.mapMany(multiples).subscribe(
  [ onNext:{ response.getWriter().println(it.toString()); },
    onCompleted:{ response.getWriter().println("Sequence complete"); },
    onError:{ response.getWriter().println("Error encountered"); } ]
);

1
2
3
2
4
6
3
6
9
Sequence complete
```

If any of the individual Observables mapped to the emissions from the source Observable in `Observable.mapMany()` aborts by calling `onError`, the `Observable.mapMany()` call itself will immediately abort and call `onError`. If you would prefer that the map-many operation continue emitting the results of the remaining, error-free Observables before reporting the error, use `Observable.mapManyDelayError()` instead.

Because it is possible for more than one of the individual Observables to encounter an error, `Observable.mapManyDelayError()` may pass information about multiple errors to the `onError` closure of its subscribers (which it will never call more than once). For this reason, if you want to know the nature of these errors, you should write your `onError` closure so that it accepts a parameter of the class `CompositeException`.

## reduce() or aggregate()
#### Apply a closure to each emitted element, sequentially, and emit only the final accumulated value

[[images/rx-operators/reduce.png]]

The `reduce()` method returns a Observable that applies a closure of your choosing to the first item emitted by a source Observable, then feeds the result of that closure along with the second item emitted by the source Observable into the same closure, then feeds the result of _that_ closure along with the third item into the same closure, and so on until all items have been emitted by the source Observable. Then it emits the final result from the final call to your closure as the sole output from the returned Observable.

This technique, which is called “reduce” or “aggregate” here, is sometimes called “fold,” “accumulate,” “compress,” or “inject” in other programming contexts. 

For example, the following code uses `reduce()` to compute, and then emit as an Observable, the sum of the numbers emitted by the source Observable:

```groovy
numbers = Observable.toObservable([1, 2, 3, 4, 5]);

Observable.reduce(numbers, { a, b -> a+b }).subscribe(
  [ onNext:{ response.getWriter().println(it); },
    onCompleted:{ response.getWriter().println("Sequence complete"); },
    onError:{ response.getWriter().println("Error encountered"); } ]
);

15
Sequence complete
```

In addition to calling `reduce()` as a stand-alone method, you can also call it as a method of a Observable object, so, in the example above, instead of 

```groovy
Observable.reduce(numbers, { a, b -> a+b }) ...
```
you could instead write 

```groovy
numbers.reduce({ a, b -> a+b }) ...
```

There is also a version of `reduce()` to which you can pass a seed value in addition to an accumulator function:

```groovy
Observable.reduce(my_observable, initial_seed, accumulator_closure)
or
my_observable.reduce(initial_seed, accumulator_closure)
```

## scan()
#### Apply a closure to each element of a sequence and emit each successive value

[[images/rx-operators/scan.png]]

The `scan()` method returns a Observable that applies a closure of your choosing to the first item emitted by a source Observable, then feeds the result of that closure along with the second item emitted by the source Observable into the same closure, then feeds the result of that closure along with the third item into the same closure, and so on until all items have been emitted by the source Observable. It emits the result of each of these iterations as a sequence from the returned Observable. This sort of closure is sometimes called an “accumulator.”

For example, the following code takes a Observable that emits a consecutive sequence of _n_ integers starting with 1 and converts it into a Observable that emits the first _n_ [triangular numbers](http://en.wikipedia.org/wiki/Triangular_number):

```groovy
numbers = Observable.toObservable([1, 2, 3, 4, 5]);

Observable.scan(numbers, { a, b -> a+b }).subscribe(
  [ onNext:{ response.getWriter().println(it); },
    onCompleted:{ response.getWriter().println("Sequence complete"); },
    onError:{ response.getWriter().println("Error encountered"); } ]
);

1
3
6
10
15
Sequence complete
```

In addition to calling `scan()` as a stand-alone method, you can also call it as a method of a Observable object, so, in the example above, instead of 

```groovy
Observable.scan(numbers, { a, b -> a+b }) ...
```
you could instead write 

```groovy
numbers.scan({ a, b -> a+b }) ...
```

There is also a version of `scan()` to which you can pass a seed value in addition to an accumulator function:

```groovy
Observable.scan(my_observable, initial_seed, accumulator_closure)
or
my_observable.scan(initial_seed, accumulator_closure)
```

**Note:** if you pass a seed value to `scan()`, it will emit the seed itself as its first value.

## groupBy()
#### divide an Observable into a set of Observables that emit groups of values from the original Observable, organized by key
The `groupBy()` method creates or extracts a key from all of the objects emitted by a source Observable. For each unique key created in this way, `groupBy()` creates an Observable that emits all of the objects from the source Observable that match that key. `groupBy()` then emits each of these Observables, as an Observable.

There are two versions of `groupBy()`:

1. One version takes two parameters: the source Observable and a closure that takes as its parameter an object emitted by the source Observable and returns the key.
1. The second version adds a third parameter: a closure that takes as its parameter an object emitted by the source Observable and returns an object to be emitted by the resulting Observable (the first version just emits the source Observable's emissions unchanged).