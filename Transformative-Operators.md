This section explains operators used for transforming elements of a sequence.

## map() & select()

#### Transform an element via a given function

[[images/operation-map.png]]

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



## mapMany()/selectMany() & mapManyDelayError()

#### Transform elements into Observables then flatten into a sequence

[[images/operation-mapMany.png]]

The `mapMany()` method creates a new Observable sequence by applying a closure that you supply to each object in the original Observable sequence, where that closure is itself a Observable that emits objects, and then merges the results of that closure applied to every item emitted by the original Observable, emitting these merged results as its own sequence.

It is useful, for example, when you have a Observable that emits a series of objects that themselves have Observable members or are in other ways transformable into Observables, so that you can create a new Observable that emits the complete collection of items emitted by the sub-Observables of these objects.

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

If any of the individual Observables mapped to the emissions from the source Observable in `Observable.mapMany()` aborts by calling `onError`, the `Observable.mapMany()` call itself will immediately abort and call `onError`. If you would prefer that the map-many continue emitting the results of the remaining, error-free Observables before reporting the error, use `Observable.mapManyDelayError()` instead.

Because it is possible that more than one of the individual observables encountered an error, `Observable.mapManyDelayError()` may pass information about multiple errors to the `onError` closure, which it will never call more than once. For this reason, if you want to know the nature of these errors, you should write the `onError` closure so that it accepts a parameter of the class `CompositeException`.




## reduce()

#### Apply a function to each element and emit the final accumulated value

[[images/operation-reduce.png]]

The `reduce()` method returns a Observable that applies a closure of your choosing to the first item emitted by a source Observable, then feeds the result of that closure along with the second item emitted by the source Observable into the same closure, then feeds the result of _that_ closure along with the third item into the same closure, and so on until all items have been emitted by the source Observable. Then it emits the final result from the final call to your closure as the sole output from the returned Observable.

This technique, which is called "reduce" here, is sometimes called "fold," "accumulate," "compress," or "inject" in other programming contexts. 

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
Observable.reduce(observable, initial_seed, accumulator_closure)
or
observable.reduce(initial_seed, accumulator_closure)
```

## scan()

#### Apply a function to each element of a sequence and emit each successive value

[[images/operation-scan.png]]

The `scan()` method returns a Observable that applies a closure of your choosing to the first item emitted by a source Observable, then feeds the result of that closure along with the second item emitted by the source Observable into the same closure, then feeds the result of that closure along with the third item into the same closure, and so on until all items have been emitted by the source Observable. It emits the result of each of these iterations as a sequence from the returned Observable. This sort of closure is sometimes called an _accumulator_.

For example, the following code takes a Observable that emits a consecutive sequence of *n* integers starting with 1 and converts it into a Observable that emits the first *n* [triangular numbers](http://en.wikipedia.org/wiki/Triangular_number):

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
Observable.scan(observable, initial_seed, accumulator_closure)
or
observable.scan(initial_seed, accumulator_closure)
```

Note that if you pass a seed value to `scan()`, it will emit the seed itself as its first value.


