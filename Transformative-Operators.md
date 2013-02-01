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
