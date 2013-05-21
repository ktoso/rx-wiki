This section explains various utility operators for working with Observables.

* **`toList()`** — collect all elements from an Observable and emit as a single List
* **`toSortedList()`** — collect all elements from an Observable and emit as a single, sorted List
* **`materialize()`** — convert an Observable into a list of Notifications
* **`dematerialize()`** — convert a materialized Observable back into its non-materialized form
* **`all()`** — determine whether all items emitted by an Observable meet some criteria
* **`finallyDo()`** — register an action to take when an Observable completes
* **`sequenceEqual()`** — determine whether two Observable sequences are identical
* **`synchronize()`** — force a poorly-behaving Observable to be well-behaved
* **`timestamp()`** — attach a timestamp to every object emitted by an Observable
* **`cache()`** — 
* **`defer()`** — 
* **`observeOn()`** — 
* **`subscribeOn()`** — 
* **`onErrorResumeNext()`** — instructs an Observable to continue emitting values after it encounters an error
* **`onErrorReturn()`** — instructs an Observable to emit a particular value when it encounters an error

## toList()
#### collect all elements from an Observable and emit as a single List

[[images/rx-operators/toList.png]]

Normally, a Observable that emits multiple items will do so by calling its observer’s `onNext` closure for each such item. You can change this behavior, instructing the Observable to compose a list of these multiple items and then to call the observer’s `onNext` closure _once_, passing it the entire list, by calling the Observable object’s `toList()` method prior to calling its `subscribe()` method. For example:

```groovy
Observable.tolist(myObservable).subscribe([ onNext: { myListOfSomething -> do something useful with the list } ]);
```

For example, the following rather pointless code takes a list of integers, converts it into a Observable, then converts that Observable into one that emits the original list as a single item:

```groovy
numbers = Observable.toObservable([1, 2, 3, 4, 5, 6, 7, 8, 9]);

Observable.toList(numbers).subscribe(
  [ onNext:{ response.getWriter().println(it); },
    onCompleted:{ response.getWriter().println("Sequence complete"); },
    onError:{ response.getWriter().println("Error encountered"); } ]
);

[1, 2, 3, 4, 5, 6, 7, 8, 9]
Sequence complete
```

In addition to calling `toList()` as a stand-alone method, you can also call it as a method of a Observable object, so, in the example above, instead of 

```groovy
Observable.toList(numbers) ...
```
 
you could instead write 

```groovy
numbers.toList() ...
```

## toSortedList()
#### collect all elements emitted by an Observable and emit this as a single sorted List

[[images/rx-operators/toSortedList.png]]

The `toSortedList()` method behaves much like `toList()` except that it sorts the resulting list. By default it sorts the list naturally in ascending order, but you can also pass in a function that takes two values and returns a number, and `toSortedList()` will use that number instead of the numerical difference between the two values to sort the values.

For example, the following code takes a list of unsorted integers, converts it into a Observable, then converts that Observable into one that emits the original list in sorted form as a single item:

```groovy
numbers = Observable.toObservable([8, 6, 4, 2, 1, 3, 5, 7, 9]);

Observable.toSortedList(numbers).subscribe(
  [ onNext:{ response.getWriter().println(it); },
    onCompleted:{ response.getWriter().println("Sequence complete"); },
    onError:{ response.getWriter().println("Error encountered"); } ]
)

[1, 2, 3, 4, 5, 6, 7, 8, 9]
Sequence complete
```

In addition to calling `toList()` as a stand-alone method, you can also call it as a method of a Observable object, so, in the example above, instead of 

```groovy
Observable.toSortedList(numbers) ...
```

you could instead write

```groovy
numbers.toSortedList( ) ...
```

## materialize()
#### convert an Observable into a list of Notifications

[[images/rx-operators/materialize.png]]

A well-formed Observable will call its observer’s `onNext` closure zero or more times, and then will call either the `onCompleted` or `onError` closure exactly once. The `Observable.materialize()` method converts this series of calls into a series of emissions from a Observable, where it represents each such call as a `Notification` object.

For example:

```groovy
numbers = Observable.toObservable([1, 2, 3]);

Observable.materialize(numbers).subscribe(
  [ onNext: { if(Kind.OnNext == it.kind) response.getWriter().println("Next: " + it.value);
              else if(Kind.OnCompleted == it.kind) response.getWriter().println("Completed");
              else if(Kind.OnError == it.kind) response.getWriter().println("Error: " + it.exception); },
    onCompleted:{ response.getWriter().println("Sequence complete"); },
    onError:{ response.getWriter().println("Error encountered"); } ]
);

Next: 1
Next: 2
Next: 3
Completed
Sequence complete
```

In addition to calling `materialize()` as a stand-alone method, you can also call it as a method of a Observable object, so that instead of 

```groovy
Observable.materialize(numbers) ...
```
in the above example, you could also write 

```groovy
numbers.materialize() ...
```

## dematerialize()
#### convert a materialized Observable back into its non-materialized form

## all()
#### determine whether all items emitted by an Observable meet some criteria

## finallyDo()
#### register an action to take when an Observable completes

## sequenceEqual()
#### determine whether two Observable sequences are identical

## synchronize()
#### force a poorly-behaving Observable to be well-behaved

## timestamp()
#### attach a timestamp to every object emitted by an Observable

## cache()
####

## defer()
####

## observeOn()
####

## subscribeOn()
####

## onErrorResumeNext()
#### instructs an Observable to attempt to continue emitting values after it encounters an error

## onErrorReturn()
#### instructs an Observable to emit a particular value to an observer’s onNext closure when it encounters an error
