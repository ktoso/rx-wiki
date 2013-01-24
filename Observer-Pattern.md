# Introduction

In the Rx (reactive) pattern an _observer_ closure _subscribes_ to an object that implements the _Observable_ interface. Then that Observer closure reacts to whatever object or sequence of objects the Observable object emits. This pattern facilitates concurrent operations by not blocking while waiting for the Observable to emit objects, but by instead creating a sentry in the form of an Observer that stands ready to react appropriately at whatever future time the Observable does so.

This page explains what the reactive pattern is, what Observables and Observers are (and how Observers subscribe to Observables), and how you use utility methods to link Observables together and change their behaviors.


> This guide accompanies its explanations with "marble diagrams." Here is how marble diagrams represent Observables and transformations of Observables:

[[images/operation-legend.png]]


# Background

In many software programming tasks, you more or less expect that the instructions you write will execute and complete incrementally, one-at-a-time, in order as you have written them. So for instance, you might write a program something like the following pseudocode:

```java
catalog       = getCatalog("Montgomery Ward");                           // get a "Montgomery Ward" catalog object
availableCash = myBankAccount.getBalance();                                // get my bank account balance
jeans         = catalog.findJeans("38W", "38L", "blue" );                 // find my size of jeans in the catalog
if( availableCash >= jeans.getPurchasePrice() ) catalog.purchase( jeans ); // if I have enough money, buy them
```

The sequence of events would, predictably and dependably, look like this: 

[[images/jeans-activity1.png]]

But in the Rx paradigm, many instructions execute in parallel and their results are later captured, in arbitrary order, by closures called "observers." In these cases, rather than _calling_ a method, you _define_ a method call in the form of a "Observable," and then _subscribe_ an observer to it, at which point the call takes place in the background with the observer standing sentry to capture and respond to its return values whenever they arrive.

An advantage of this approach is that when you have a bunch of tasks to do that are not dependent on each other, you can start them all at the same time rather than waiting for each one to finish before starting the next one --- that way, your entire bundle of tasks only takes as long to complete as the longest task in the bundle.

An important part of your design process will be to analyze your tasks with an eye to which ones can happen asynchronously and in parallel, and which ones must block because of dependencies on the results of other tasks.

Here is how the equivalent jeans-buying process might take place in the reactive model:
```groovy
catalogObservable = getCatalog("Montgomery Ward");
catalogObservable.mapMany( catalog -> catalog.findJeans("38W", "38L", "blue" ) )
                .zip( myBankAccount.getBalance( ) ) { product, cash -> if( cash > product.getPurchasePrice() ) product.purchase() ) };
```

And rather than a sequence diagram, a marble diagram will be used to illustrate its behavior:

[[images/jeans-marble1.png]]

> There are many terms used to describe this model of asynchronous programming and design. This document will use the following terms: An _observer_ is a closure that you _subscribe_ to an object that implements the _Observable_ interface; that is, you _subscribe_ an _observer_ to an _Observable_.

> In other documents and other contexts, what we are calling a "observer" is sometimes called a "reactor." This model in general is often referred to as the ["reactor pattern"|http://en.wikipedia.org/wiki/Reactor_pattern].


# Setting up Observers

> Groovy will be used for code examples in this document but any JVM language can be used such as Clojure, Scala, JRuby, Javascript or Java itself. 
> For Java itself, anywhere 'closure' is mentioned consider it an anonymous inner class.

In an ordinary method call --- that is, _not_ the sort of asynchronous, parallel calls typical in reactive programming --- the flow is something like this:

1) Call a method.
2) Store the return value from that method in a variable.
3) Use that variable and its new value to do something useful.

Or, something like this:

```groovy
returnVal = someMethod( *itsParameters* ); ⇒ make the call, assign its return value to `returnVal`
*do something useful with* returnVal;
```

In the asynchronous model the flow goes more like this:

1) Define a closure that does something useful with the return value from the asynchronous call, this is called the _observer_.
2) Define the asynchronous call itself as an object that inherits from [Observable|http://netflix.github.com/RxJava/rx/observables/Observable.html].
3) Attach the observer to that Observable asynchronous call by subscribing it (this also initiates the call).
4) Go on with your business; whenever the call returns, the _observer_ will begin to operate on its return value.

Which looks something like this:

```groovy
def myObserver = \{ it \-> *do something useful with* it \}; ⇒ defines, but does not invoke, the observer
def myObservable = *someObservableMethod*( *itsParameters* ); ⇒ defines, but does not invoke, the Observable
myObservable.subscribe(\[ onNext:myObserver \]); ⇒ subscribes the observer to the Observable, and invokes the Observable
*go on about my business*
```

## `onNext`, `onCompleted`, and `onError`

You will notice that the `subscribe( )` method does not take a simple closure as its parameter, but a Map. In the example above, this maps "`onNext`" onto the closure defined as `myObserver`.

`onNext` is a keyword particular to `subscribe()`. It defines the closure that the Observable will invoke whenever the Observable emits a value.

You can also pass in to the `subscribe()` method two additional closures that the Observable will invoke at different times:

| `onCompleted` | A Observable will invoke this closure after it has called `onNext` for the final time if it has not encountered any errors. |
| `onError^ | A Observable will invoke this closure to indicate that it has failed to generate the expected data. By default this stops the Observable and it will not make further calls to `onNext` or `onCompleted`. |

A more complete `subscribe()` example would therefore look like this:

```groovy
def myObserver   = \{ it \-> *do something useful with* it \};
def myComplete  = \{ *clean up after the final response* \};
def myError     = \{ exception \-> *react sensibly to a failed call* \};
def myObservable = *someMethod*( *itsParameters* );
myObservable.subscribe(\[ onNext:myObserver, onCompleted:myComplete, onError:myError \]);
*go on about my business*
```

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

[[images/marble.just.png]]

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
