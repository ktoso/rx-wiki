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