# Introduction

In the Rx (reactive) pattern an _observer_ closure _subscribes_ to an object that implements the _Observable_ interface. Then that Observer closure reacts to whatever object or sequence of objects the Observable object emits. This pattern facilitates concurrent operations by not blocking while waiting for the Observable to emit objects, but by instead creating a sentry in the form of an Observer that stands ready to react appropriately at whatever future time the Observable does so.

This page explains what the reactive pattern is and what Observables and Observers are (and how Observers subscribe to Observables). Subsequent child pages (shown in sidebar) show how you use utility methods on the Observable class to link Observables together and change their behaviors.

> This documentation accompanies its explanations with "marble diagrams." Here is how marble diagrams represent Observables and transformations of Observables:

[[images/operation-legend.png]]

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

The sequence of events would, predictably and dependably, look like this: 

**missing image**
[[images/jeans-activity1.png]]

But in the Rx paradigm, many instructions execute in parallel and their results are later captured, in arbitrary order, by closures called "observers." In these cases, rather than _calling_ a method, you _define_ a method call in the form of an "Observable," and then _subscribe_ an "Observer" to it, at which point the call takes place in the background with the observer standing sentry to capture and respond to its return values whenever they arrive.

An advantage of this approach is that when you have a bunch of tasks to do that are not dependent on each other, you can start them all at the same time rather than waiting for each one to finish before starting the next one --- that way, your entire bundle of tasks only takes as long to complete as the longest task in the bundle.

Here is how the equivalent jeans-buying process might take place in the reactive model:
```groovy
catalogObservable = getCatalog("Montgomery Ward");
catalogObservable
   .mapMany({catalog -> catalog.findJeans("38W", "38L", "blue" )})
   .zip(myBankAccount.getBalance(),
        {product, cash -> if(cash > product.getPurchasePrice()) product.purchase() });
```

And rather than a sequence diagram, a marble diagram will be used to illustrate its behavior:

**missing image**
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
// make the call, assign its return value to `returnVal`
returnVal = someMethod(itsParameters);
// do something useful with returnVal
```

In the asynchronous model the flow goes more like this:

1) Define a closure that does something useful with the return value from the asynchronous call, this is called the _observer_.  
2) Define the asynchronous call itself as an object that inherits from [Observable|http://netflix.github.com/RxJava/rx/observables/Observable.html].  
3) Attach the observer to that Observable asynchronous call by subscribing it (this also initiates the call).  
4) Go on with your business; whenever the call returns, the _observer_ will begin to operate on its return value.  

Which looks something like this:

```groovy
// defines, but does not invoke, the observer
def myObserver = { it -> do something useful with it };
// defines, but does not invoke, the Observable
def myObservable = someObservableMethod(itsParameters);
// subscribes the observer to the Observable, and invokes the Observable
myObservable.subscribe([ onNext:myObserver ]);
// go on about my business
```

## onNext, onCompleted, and onError

You will notice that the `subscribe()` method does not take a simple closure as its parameter, but a Map. In the example above, this maps `onNext` onto the closure defined as `myObserver`.

`onNext` is a keyword particular to `subscribe()`. It defines the closure that the Observable will invoke whenever the Observable emits a value.

You can also pass in to the `subscribe()` method two additional closures that the Observable will invoke at different times:

### onCompleted

An Observable will invoke this closure after it has called `onNext` for the final time if it has not encountered any errors.

### onError

An Observable will invoke this closure to indicate that it has failed to generate the expected data. By default this stops the Observable and it will not make further calls to `onNext` or `onCompleted`.

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

The Observable/Observer classes along with onNext/onError/onCompleted are only the start of Rx. By themselves they'd be nothing more than a slight extension of the standard observer pattern better suited to handling a sequence of events rather than a single callback.

The real power comes with the "reactive extensions" (hence "Rx") that are made up by the various operators that allow transforming, combining, manipulating and working with sequences of asynchronous data exposed through Observables.

The Rx operators allow composing asynchronous sequences together in a declarative manner with all the efficiency benefits of callbacks but without the drawbacks of nesting callback handlers typically associated with asynchronous systems.

Information about the various operators and examples of their usage has been grouped into the following pages (also listed in the sidebar):

* [[Combinatorial|Observable-Operators-Combinatorial]]
* [[Transformative|Observable-Operators-Transformative]]
* [[Filtering|Observable-Operators-Filtering]]
* [[Creation|Observable-Operators-Creation]]
* [[Error Handling|Observable-Operators-ErrorHandling]]
* [[Utility|Observable-Operators-Utility]]