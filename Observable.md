In RxJava an object of the _Subscriber_ class (or some other object that implements the _Observer_ interface) _subscribes_ to an object of the _Observable_ class. Then that subscriber reacts to whatever item or items the Observable object _emits_. This pattern facilitates concurrent operations because it does not need to block while waiting for the Observable to emit objects, but instead it creates a sentry in the form of a subscriber that stands ready to react appropriately at whatever future time the Observable does so.

This page explains what the reactive pattern is and what Observables and Subscribers are (and how Subscribers subscribe to Observables). Subsequent child pages (as shown in sidebar) show how you use the variety of Observable operators to link Observables together and change their behaviors.

> This documentation accompanies its explanations with "marble diagrams." Here is how marble diagrams represent Observables and transformations of Observables:

[[images/rx-operators/legend.png]]

#### see also
* <a href="http://channel9.msdn.com/Series/Rx-Workshop/Rx-Workshop-Introduction">Rx Workshop: Introduction</a>
* <a href="http://www.introtorx.com/Content/v1.0.10621.0/02_KeyTypes.html#IObservable">Introduction to Rx: IObservable</a>

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

But in the Rx paradigm, many instructions execute in parallel and their results are later captured, in arbitrary order, by “Subscribers.” In these cases, rather than _calling_ a method, you define a mechanism for retrieving and transforming the data, in the form of an “Observable,” and then _subscribe_ an “Subscriber” to it, at which point the previously-defined mechanism fires into action with the Subscriber standing sentry to capture and respond to its emissions whenever they arrive.

An advantage of this approach is that when you have a bunch of tasks to do that are not dependent on each other, you can start them all at the same time rather than waiting for each one to finish before starting the next one --- that way, your entire bundle of tasks only takes as long to complete as the longest task in the bundle.

Here is how the equivalent jeans-buying process might take place in the reactive model:
```groovy
// define the mechanism for retrieving the catalog as an Observable that emits catalogs
catalogObservable = getCatalog("Montgomery Ward");
// define how to transform a catalog into an Observable that emits a particular jeans record...
catalogObservable
   .mapMany({catalog -> catalog.findJeans("38W", "38L", "blue" )})
// ...combine the results of this with the emission from an Observable that retrieves my bank balance...
   .zip(myBankAccount.getBalance(),
// ...and transform this combination by ordering the jeans if the price is right...
        {product, cash -> if(cash > product.getPurchasePrice()) product.purchase() });
```

> There are many terms used to describe this model of asynchronous programming and design. This document will use the following terms: A _Subscriber_ (or sometimes _Observer_) _subscribes_ to an object of the _Observable_ class; that is, you _subscribe_ a _Subscriber_ to an _Observable_. An Observable _emits_ _items_ or sends _notifications_ to its Subscribers by invoking the Subscribers’ methods.

> In other documents and other contexts, what we are calling a “Subscriber” (or sometimes “Observer”) is sometimes called a “watcher” or “reactor.” This model in general is often referred to as the [“reactor pattern”](http://en.wikipedia.org/wiki/Reactor_pattern).

# Establishing Subscribers

> This document usually uses Groovy for code examples, but you can use RxJava in any JVM language — such as Clojure, Scala, JRuby, Javascript, or in Java itself.  

In an ordinary method call — that is, _not_ the sort of asynchronous, parallel calls typical in reactive programming — the flow is something like this:

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

1. Define a method that does something useful with the return value from the asynchronous call, this method is part of the _Subscriber_.  
1. Define the asynchronous call itself as an object of the [Observable](http://netflix.github.com/RxJava/javadoc/rx/Observable.html) class.  
1. Attach the Subscriber to that Observable by _subscribing_ it (this also initiates the method call).  
1. Go on with your business; whenever the call returns, the Subscriber’s method will begin to operate on its return value or values — the _items_ emitted by the Observable.  

Which looks something like this:

```groovy
// defines, but does not invoke, the Subscriber's onNext handler
// (in this example, the Subscriber is very simple and has only an onNext handler)
def myOnNext = { it -> do something useful with it };
// defines, but does not invoke, the Observable
def myObservable = someObservable(itsParameters);
// subscribes the Subscriber to the Observable, and invokes the Observable
myObservable.subscribe(myOnNext);
// go on about my business
```

## onNext, onCompleted, and onError

The `subscribe()` method may accept one to three methods, or it may accept a `Subscriber` object or any other object that implements the `Observer` interface (which includes these three methods):

**onNext**: An Observable will call this methods of its Subscribers whenever the Observable emits an item. This method takes as a parameter the item emitted by the Observable.

**onError**: An Observable will call this method of its Subscribers to indicate that it has failed to generate the expected data. This stops the Observable and it will not make further calls to `onNext` or `onCompleted`. The `onError` method takes as its parameter the Throwable that caused the error (or a `CompositeException` in those cases where there may have been multiple errors).

**onCompleted**: An Observable will invoke this method of its Subscribers after it has called `onNext` for the final time, if it has not encountered any errors.

A more complete `subscribe()` example would therefore look like this:

```groovy
def myOnNext     = { it -> /* do something useful with it */ };
def myError      = { Throwable -> /* react sensibly to a failed call */ };
def myComplete   = { /* clean up after the final response */ };
def myObservable = someMethod(itsParameters);
myObservable.subscribe(myOnNext, myError, myComplete);
// go on about my business
```

#### see also:
* <a href="http://www.introtorx.com/Content/v1.0.10621.0/02_KeyTypes.html#IObserver">Introduction to Rx: IObserver</a>

## Unsubscribing

An object of the `Subscriber` class also implements the `unsubscribe()` method (this method is not part of the `Observer` interface, so objects that implement that interface may or may not implement this method). You can call this method to indicate that the Subscriber is no longer interested in any of the Observables it is currently subscribed to. Those Observables can then (if they have no other interested subscribers) choose to stop generating new items to emit.

The results of this unsubscription will cascade back through the chain of operators that applies to the Observable that the Subscriber subscribed to, and this will cause each link in the chain to stop emitting items. This is not guaranteed to happen immediately, however, and it is possible for an Observable to generate and attempt to emit items for a while even after no Subscribers remain to observe these emissions.

## Some Notes on Naming Conventions

The names of methods and classes in RxJava hew close to those in <a href="http://msdn.microsoft.com/en-us/data/gg577609.aspx">Microsoft’s Reactive Extensions</a>. This has led to some confusion, as some of these names have different implications in other contexts, or seem awkward in the idiom of a particular implementing language.

For example there is the `onEvent` naming pattern (e.g. `onNext`, `onCompleted`, `onError`). In many contexts such names would indicate methods by means of which event handlers are _registered_. In the RxJava Subscriber context, however, they name the event handlers themselves.

# Composition via Observable Operators

The Observable/Subscriber classes along with onNext/onError/onCompleted are only the start of RxJava. By themselves they’d be nothing more than a slight extension of the standard observer pattern, better suited to handling a sequence of events rather than a single callback.

The real power comes with the “reactive extensions” (hence “Rx”) — operators that allow you to transform, combine, manipulate, and work with the sequences of items emitted by Observables.

These Rx operators allow you to compose asynchronous sequences together in a declarative manner with all the efficiency benefits of callbacks but without the drawbacks of nesting callback handlers that are typically associated with asynchronous systems.

This documentation groups information about the various operators and examples of their usage into the following pages (these are also listed in the sidebar):

  * [[Creating|Creating-Observables]]
  * [[Transforming|Transforming-Observables]]
  * [[Filtering|Filtering-Observables]]
  * [[Combining|Combining-Observables]]
  * [[Error Handling|Error-Handling-Operators]]
  * [[Utility|Observable-Utility-Operators]]
  * [[Conditional and Boolean|Conditional-and-Boolean-Operators]]
  * [[Mathematical and Aggregate|Mathematical-and-Aggregate-Operators]]
  * [[Asynchronous Conversion|Async-Operators]]
  * [[Connectable Observables|Connectable-Observable-Operators]]
  * [[Blocking Observables|Blocking-Observable-Operators]]
  * [[String Observables|String-Observables]]

## Implementing Your Own Operators with lift()

You can implement your own RxJava operators and then chain them along with those that are already part of RxJava. To do this, define your operator as a public class, like so:
```java
public class myOperator<T> implements Operator<T> {
  public myOperator( /* any necessary params here */ ) {
    /* any necessary initialization here */
  }

  @Override
  public Subscriber<? super T> call(final Subscriber<? super T> s) {
    return new Subscriber<t>(s) {
      @Override
      public void onCompleted() {
        /* add your own onCompleted behavior here, or just pass the completed notification through: */
        s.onCompleted();
      }

      @Override
      public void onError(Throwable t) {
        /* add your own onError behavior here, or just pass the error notification through: */
        s.onError(t);
      }

      @Override
      public void onNext(T item) {
        /* this example performs some sort of simple transformation on each incoming item and then passes it along */
        if(!s.isUnsubscribed()) {
          transformedItem = myOperatorTransformOperation(item);
          s.onNext(transformedItem);
        }
      }
    };
  }
}
``` 
You can then add your custom operator to a chain of operators operating on an Observable by using the `lift()` operator, for example:
```groovy
Observable foo = barObservable.ofType(Integer).map({it*2}).lift(new myOperator<T>()).map({"transformed by myOperator: " + it});
```