# Creating Observables

This section explains how to create Observables either explicitly or by wrapping an existing data structure.

## toObservable() & from()

#### Make an Observable from an Iterable.

[[images/operation-toObservable.png]]

Any object that supports the `Iterable<>` interface can be converted into a Observable that emits each iterable item in the object, simply by passing the object into the `Observable.toObservable( )` method, for example:

```groovy
myObservable = Observable.toObservable(myIterable);
```

You can also do this with arrays, for example:

```groovy
myArray = [1, 2, 3, 4, 5];
myArrayObservable = Observable.toObservable(myArray);
```

This converts the sequence of values in the iterable object or array into a sequence of objects emitted, one at a time, by a Observable.

## Observable.just()

#### Make any object into a Observable.

[[images/operation-just.png]]

To convert any object into a Observable that emits that object, pass that object into the `Observable.just()` method.

```groovy
// Observable emits "some string" as a single item
def observableThatEmitsAString = Observable.just("some string"); 
// Observable emits the list [1, 2, 3, 4, 5] as a single item
def observableThatEmitsAList = Observable.just([1, 2, 3, 4, 5]); 
```

This is similar to the `Observable.toObservable()` method, except that `Observable.toObservable()` will convert an iterable object into a Observable that emits each of the items in the iterable, one at a time, while the `Observable.just()` method would convert the iterable into a Observable that emits the entire iterable as a single item.

## Observable.create( )

#### Create an explicit Observable

You can create an Observable from scratch, by using the `Observable.create()` method. You pass this method a closure that accepts as a parameter the Observer that is passed to a Observable’s `subscribe()` method. Write the closure you pass to `Observable.create()` so that it behaves as an Observable --- calling the passed-in `onNext`, `onError`, and `onCompleted` methods appropriately. For example:

```groovy
def myObservable = Observable.create({ m ->
  m.onNext('One');
  m.onNext('Two');
  m.onNext('Three');
  m.onNext('Four');
  m.onCompleted();
})
```

NOTE: A well-formed Observable _must_ call either the observer’s `onCompleted()` method exactly once or its `onError()` method exactly once.