# Creating Observables

This section explains methods that create Observables.

* **`toObservable()` or `from()`** — convert an Iterable into an Observable
* **`just()`** — convert an object into an Observable that emits that object
* **`create()`** — create an Observable from scratch by means of a closure
* **`range()`** — create an Observable that emits a range of sequential integers
* **`empty()`** — create an Observable that emits nothing and then calls `onCompleted()` 
* **`error()`** — create an Observable that emits nothing and then calls `onError()` 
* **`never()`** — create an Observable that emits nothing at all

## toObservable() & from()
#### convert an Iterable into an Observable

[[images/rx-operators/toObservable.png]]

Any object that supports the `Iterable<>` interface can be converted into a Observable that emits each iterable item in the object, simply by passing the object into the `toObservable( )` method, for example:

```groovy
myObservable = Observable.toObservable(myIterable);
```

You can also do this with arrays, for example:

```groovy
myArray = [1, 2, 3, 4, 5];
myArrayObservable = Observable.toObservable(myArray);
```

This converts the sequence of values in the iterable object or array into a sequence of objects emitted, one at a time, by a Observable.

## just()
#### convert an object into an Observable that emits that object

[[images/rx-operators/just.png]]

To convert any object into a Observable that emits that object, pass that object into the `just()` method.

```groovy
// Observable emits "some string" as a single item
def observableThatEmitsAString = Observable.just("some string"); 
// Observable emits the list [1, 2, 3, 4, 5] as a single item
def observableThatEmitsAList = Observable.just([1, 2, 3, 4, 5]); 
```

This is similar to the `toObservable()` method, except that `toObservable()` will convert an iterable object into a Observable that emits each of the items in the iterable, one at a time, while the `just()` method would convert the iterable into a Observable that emits the entire iterable as a single item.

## create( )
#### create an Observable from scratch by means of a closure

[[images/rx-operators/create.png]]

You can create an Observable from scratch, by using the `create()` method. You pass this method a closure that accepts as a parameter the Observer that is passed to a Observable’s `subscribe()` method. Write the closure you pass to `create()` so that it behaves as an Observable --- calling the passed-in Observer’s `onNext()`, `onError()`, and `onCompleted()` methods appropriately. For example:

```groovy
def myObservable = Observable.create({ anObserver ->
  anObserver.onNext('One');
  anObserver.onNext('Two');
  anObserver.onNext('Three');
  anObserver.onNext('Four');
  anObserver.onCompleted();
})
```

**NOTE:** A well-formed Observable _must_ call either the observer’s `onCompleted()` method exactly once or its `onError()` method exactly once.

## `range()`
#### create an Observable that emits a range of sequential integers

## `empty()`, `error()`, and `never()`
#### Observables that can be useful for testing purposes