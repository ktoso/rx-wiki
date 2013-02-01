This section explains operators used for combining sequences together.

## concat()

#### Concatenate two or more Observables sequentially

[[images/operation-concat.png]]

You can concatenate the output of multiple Observables so that they act like a single Observable, with all of the items emitted by the first Observable being emitted before any of the items emitted by the second Observable, by using the `Observable.concat()` method:

```groovy
myConcatenatedObservable = Observable.concat(observable1, observable2, … );
```

For example, the following code concatenates the 'odds' and 'evens' Observables into a single Observable:

```groovy
odds  = Observable.toObservable([1, 3, 5, 7]);
evens = Observable.toObservable([2, 4, 6]);

Observable.concat(odds, evens).subscribe(
  [ onNext:{ response.getWriter().println(it); },
    onCompleted:{ response.getWriter().println("Sequence complete"); },
    onError:{ response.getWriter().println("Error encountered"); } ]
)

1
3
5
7
2
4
6
Sequence complete
```

Instead of passing multiple Observables into `Observable.concat()`, you could also pass in a `List<>` of Observables, or even a Observable that emits Observables, and `Observable.concat()` will concatenate their output into the output of a single Observable.
