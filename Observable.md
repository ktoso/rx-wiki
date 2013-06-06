# Introduction

In the Rx (reactive) pattern an _Observer_ object _subscribes_ to an object that implements the _Observable_ interface. Then that Observer reacts to whatever item or items the Observable object emits. This pattern facilitates concurrent operations by not blocking while waiting for the Observable to emit objects, but by instead creating a sentry in the form of an Observer that stands ready to react appropriately at whatever future time the Observable does so.

This page explains what the reactive pattern is and what Observables and Observers are (and how Observers subscribe to Observables). Subsequent child pages (as shown in sidebar) show how you use utility methods on the Observable class to link Observables together and change their behaviors.

> This documentation accompanies its explanations with "marble diagrams." Here is how marble diagrams represent Observables and transformations of Observables:

[[images/rx-operators/legend.png]]

# Background

In many software programming tasks, you more or less expect that the instructions you write will execute and complete incrementally, one-at-a-time, in order as you have written them. So for instance, you might write a program something like the following pseudocode:

```java
// get a "Montgomery Ward" catalog object
catalog       = getCatalog("Montgomery Ward");
// get my bank account balance
availableCash = myBankAccount.getBalance();
// find my size of jeans in the catalog
jeans         = catalog.findJeans("38W", "38L", "blue" ); 
// if I have enough money, buy them
if( availableCash >= jeans.getPurchasePrice() ) catalog.purchase( jeans );
```

But in the Rx paradigm, many instructions execute in parallel and their results are later captured, in arbitrary order, by “Observers.” In these cases, rather than _calling_ a method, you _define_ a method call in the form of an “Observable,” and then _subscribe_ an “Observer” to it, at which point the call takes place in the background with the Observer standing sentry to capture and respond to its return values whenever they arrive.

An advantage of this approach is that when you have a bunch of tasks to do that are not dependent on each other, you can start them all at the same time rather than waiting for each one to finish before starting the next one --- that way, your entire bundle of tasks only takes as long to complete as the longest task in the bundle.

Here is how the equivalent jeans-buying process might take place in the reactive model:
```groovy
catalogObservable = getCatalog("Montgomery Ward");
catalogObservable
   .mapMany({catalog -> catalog.findJeans("38W", "38L", "blue" )})
   .zip(myBankAccount.getBalance(),
        {product, cash -> if(cash > product.getPurchasePrice()) product.purchase() });
```

> There are many terms used to describe this model of asynchronous programming and design. This document will use the following terms: An _Observer_ object _subscribes_ to an object that implements the _Observable_ interface; that is, you _subscribe_ an _Observer_ to an _Observable_. An Observable _emits_ _items_ or sends _notifications_ to its Observers by invoking the Observers' methods.

> In other documents and other contexts, what we are calling a “Observer” is sometimes called a “watcher” or “reactor.” This model in general is often referred to as the [“reactor pattern”](http://en.wikipedia.org/wiki/Reactor_pattern).

# Setting up Observers

> This document uses Groovy for code examples, but you can use RxJava in any JVM language --- such as Clojure, Scala, JRuby, Javascript, or Java itself.  

In an ordinary method call --- that is, _not_ the sort of asynchronous, parallel calls typical in reactive programming --- the flow is something like this:

1. Call a method.  
1. Store the return value from that method in a variable.  
1. Use that variable and its new value to do something useful.  

Or, something like this:

```groovy
// make the call, assign its return value to `returnVal`
returnVal = someMethod(itsParameters);
// do something useful with returnVal
```

In the asynchronous model the flow goes more like this:

1. Define a method that does something useful with the return value from the asynchronous call, this method is part of the _Observer_.  
1. Define the asynchronous call itself as an [Observable](http://netflix.github.com/RxJava/javadoc/rx/Observable.html) object.  
1. Attach the Observer to that Observable by _subscribing_ it (this also initiates the call).  
1. Go on with your business; whenever the call returns, the Observer's method will begin to operate on its return value or values --- the _items_ emitted by the Observable.  

Which looks something like this:

```groovy
// defines, but does not invoke, the observer
def myObserver = { it -> do something useful with it };
// defines, but does not invoke, the Observable
def myObservable = someObservable(itsParameters);
// subscribes the observer to the Observable, and invokes the Observable
myObservable.subscribe([ onNext:myObserver ]);
// go on about my business
```

## onNext, onCompleted, and onError

You will notice that the `subscribe()` method does not take a simple method as its parameter, but a Map. In the example above, this maps `onNext` onto the method defined as `myObserver`.

`onNext` is a keyword particular to `subscribe()`. It defines the method that the Observable will invoke whenever the Observable emits an item. This method takes as a parameter an item emitted by the Observable.

You can also pass in to the `subscribe()` method two additional Observer methods that the Observable will invoke at different times: `onCompleted` and `onError`.

**onCompleted**: An Observable will invoke this Observer method after it has called `onNext` for the final time, if it has not encountered any errors.

**onError**: An Observable will invoke this Observer method to indicate that it has failed to generate the expected data. By default this stops the Observable and it will not make further calls to `onNext` or `onCompleted`. The `onError` method takes as its parameter the Exception that caused the error (or a `CompositeException` in those cases where there may have been multiple exceptions).

A more complete `subscribe()` example would therefore look like this:

```groovy
def myObserver   = { it -> do something useful with it };
def myComplete  = { clean up after the final response };
def myError     = { exception -> react sensibly to a failed call };
def myObservable = someMethod(itsParameters);
myObservable.subscribe([ onNext:myObserver, onCompleted:myComplete, onError:myError ]);
// go on about my business
```

# Composition via Observable Operators

The Observable/Observer classes along with onNext/onError/onCompleted are only the start of RxJava. By themselves they'd be nothing more than a slight extension of the standard observer pattern, better suited to handling a sequence of events rather than a single callback.

The real power comes with the “reactive extensions” (hence “Rx”) --- operators that allow you to transform, combine, manipulate, and work with the sequences of asynchronous items emitted by Observables.

These Rx operators allow you to compose asynchronous sequences together in a declarative manner with all the efficiency benefits of callbacks but without the drawbacks of nesting callback handlers that are typically associated with asynchronous systems.

This documentation groups information about the various operators and examples of their usage into the following pages (these are also listed in the sidebar):

  * [[Creating|Creating-Observables]]
  * [[Transforming|Transforming-Observables]]
  * [[Filtering|Filtering-Observables]]
  * [[Combining|Combining-Observables]]
  * [[Utility|Observable-Utility-Operators]]
  * [[Connectable Observables|Connectable-Observable-Operators]]
  * [[Blocking Observables|Blocking-Observable-Operators]]