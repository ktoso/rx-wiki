# Introduction

In the Rx (reactive) pattern an _observer_ closure _subscribes_ to an object that implements the _Observable_ interface. Then that Observer closure reacts to whatever object or sequence of objects the Observable object emits. This pattern facilitates concurrent operations by not blocking while waiting for the Observable to emit objects, but by instead creating a sentry in the form of an Observer that stands ready to react appropriately at whatever future time the Observable does so.

This page explains what the reactive pattern is, what Observables and Observers are (and how Observers subscribe to Observables), and how you use utility methods to link Observables together and change their behaviors.

```
This guide accompanies its explanations with "marble diagrams." Here is how marble diagrams represent Observables and transformations of Observables:
<img src="images/marble-legend.png" width="100" height"100">
```

# Background

In many software programming tasks, you more or less expect that the instructions you write will execute and complete incrementally, one-at-a-time, in order as you have written them. So for instance, you might write a program something like the following pseudocode:

```java
catalog       = getCatalog( 'Montgomery Ward' );                           // get a "Montgomery Ward" catalog object
availableCash = myBankAccount.getBalance();                                // get my bank account balance
jeans         = catalog.findJeans( '38W', '38L', 'blue' );                 // find my size of jeans in the catalog
if( availableCash >= jeans.getPurchasePrice() ) catalog.purchase( jeans ); // if I have enough money, buy them
```

The sequence of events would, predictably and dependably, look like this: 

<img src="images/jeans-activity1.png" width="100" height"100">

But in the Rx paradigm, many instructions execute in parallel and their results are later captured, in arbitrary order, by closures called "observers." In these cases, rather than _calling_ a method, you _define_ a method call in the form of a "Observable," and then _subscribe_ an observer to it, at which point the call takes place in the background with the observer standing sentry to capture and respond to its return values whenever they arrive.

An advantage of this approach is that when you have a bunch of tasks to do that are not dependent on each other, you can start them all at the same time rather than waiting for each one to finish before starting the next one --- that way, your entire bundle of tasks only takes as long to complete as the longest task in the bundle.

An important part of your design process will be to analyze your tasks with an eye to which ones can happen asynchronously and in parallel, and which ones must block because of dependencies on the results of other tasks.

Here is how the equivalent jeans-buying process might take place in the reactive model:
```groovy
catalogObservable = getCatalog( 'Montgomery Ward' );
catalogObservable.mapMany( catalog -> catalog.findJeans( '38W', '38L', 'blue' ) )
                .zip( myBankAccount.getBalance( ) ) { product, cash -> if( cash > product.getPurchasePrice() ) product.purchase() ) };
```

And rather than a sequence diagram, a marble diagram will be used to illustrate its behavior:

<img src="images/jeans-marble1.png" width="100" height"100">

```
There are many terms used to describe this model of asynchronous programming and design. This document will use the following terms: An _observer_ is a closure that you _subscribe_ to an object that implements the _Observable_ interface; that is, you _subscribe_ an _observer_ to an _Observable_.

In other documents and other contexts, what we are calling a "observer" is sometimes called a "reactor." This model in general is often referred to as the ["reactor pattern"|http://en.wikipedia.org/wiki/Reactor_pattern].
```