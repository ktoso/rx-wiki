# Creating Observables

When you use the various Observable utility functions to chain Observables together, you will occasionally want to convert other objects into Observables so that they can join in the chain. The methods explained in this section are simple ways to accomplish this.

## Making any iterable into a Observable --- `Observable.toObservable( )`

[[images/operation-toObservable.png]]

Any object that supports the `Iterable<>` interface can be converted into a Observable that emits each iterable item in the object, simply by passing the object into the `Observable.toObservable( )` method, for example:

```groovy
myObservable = Observable.toObservable( myIterable );
```

You can also do this with arrays, for example:

```groovy
myArray = [ 1, 2, 3, 4, 5 ];
myArrayObservable = Observable.toObservable( myArray );
```

This converts the sequence of values in the iterable object or array into a sequence of objects emitted, one at a time, by a Observable.

## Making any object into a Observable --- `Observable.just( )`

[[images/operation-just.png]]

To convert any object into a Observable that emits that object, pass that object into the `Observable.just()` method.

```groovy
def observableThatEmitsAString = Observable.just( "some string" ); // Observable emits "some string" as a single item
def observableThatEmitsAList = Observable.just( [1, 2, 3, 4, 5] ); // Observable emits the list [1, 2, 3, 4, 5] as a single item
```

This is similar to the `Observable.toObservable( )` method, except that `Observable.toObservable( )` will convert an iterable object into a Observable that emits each of the items in the iterable, one at a time, while the `Observable.just( )` method would convert the iterable into a Observable that emits the entire iterable as a single item.

## Creating a Observable --- `Observable.create( )`

You can create a simple Observable from scratch, by using the `Observable.create( )`|http://go/apidoc/com/netflix/api/platform/reactive/operations/ObservableExtensions.html#create(com.netflix.api.platform.functions.Func1)]^ method. You pass this method a closure that accepts as a parameter the map of closures that a observer passes to a Observable’s `subscribe( )` method. Write the closure you pass to `Observable.create( )` so that it behaves as a Observable --- calling the passed-in `onNext`, `onError`, and `onCompleted` methods appropriately. For example:

```groovy
def myObservable = Observable.create({ m ->
  m.onNext('One');
  m.onNext('Two');
  m.onNext('Three');
  m.onNext('Four');
  m.onCompleted();
})
```
{note}A well-formed Observable _must_ call either the observer’s `onCompleted( )` method exactly once or its `onError( )` method exactly once.{note}
