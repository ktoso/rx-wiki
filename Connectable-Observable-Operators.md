This section explains the [`ConnectableObservable`](http://netflix.github.io/RxJava/javadoc/rx/observables/ConnectableObservable.html) subclass and its operators:

* [**`ConnectableObservable.connect( )`**](Connectable-Observable-Operators#connectableobservableconnect) — instructs a Connectable Observable to begin emitting items
* [**`Observable.publish( )` and `Observable.multicast( )`**](Connectable-Observable-Operators#observablepublishandobservablemulticast) — represents an Observable as a Connectable Observable
* [**`Observable.replay( )`**](Connectable-Observable-Operators#observablereplay) — ensures that all Observers see the same sequence of emitted items, even if they subscribe after the Observable begins emitting the items

A Connectable Observable resembles an ordinary Observable, except that it does not begin emitting items when it is subscribed to, but only when its `connect()` method is called. In this way you can wait for all intended Observers to subscribe to the Observable before the Observable begins emitting items.

[[images/rx-operators/publishConnect.png]]

The following example code shows two Observers subscribing to the same Observable. In the first case, they subscribe to an ordinary Observable; in the second case, they subscribe to a Connectable Observable that only connects after both observers subscribe. Note the difference in the output:

**Example #1:**
```groovy
def firstMillion  = Observable.range( 1, 1000000 ).sample(7, java.util.concurrent.TimeUnit.MILLISECONDS);

firstMillion.subscribe(
  [ onNext:{ myWriter.println("Observer #1:" + it); },
    onCompleted:{ myWriter.println("Sequence #1 complete"); },
    onError:{ myWriter.println("Error encountered"); } ]
);

firstMillion.subscribe(
  [ onNext:{ myWriter.println("Observer #2:" + it); },
    onCompleted:{ myWriter.println("Sequence #2 complete"); },
    onError:{ myWriter.println("Error encountered"); } ]
);
```
```
Observer #1:211128
Observer #1:411633
Observer #1:629605
Observer #1:841903
Sequence #1 complete
Observer #2:244776
Observer #2:431416
Observer #2:621647
Observer #2:826996
Sequence #2 complete
```
**Example #2:**
```groovy
def firstMillion  = Observable.range( 1, 1000000 ).sample(7, java.util.concurrent.TimeUnit.MILLISECONDS).publish();

firstMillion.subscribe(
  [ onNext:{ myWriter.println("Observer #1:" + it); },
    onCompleted:{ myWriter.println("Sequence #1 complete"); },
    onError:{ myWriter.println("Error encountered"); } ]
);

firstMillion.subscribe(
  [ onNext:{ myWriter.println("Observer #2:" + it); },
    onCompleted:{ myWriter.println("Sequence #2 complete"); },
    onError:{ myWriter.println("Error encountered"); } ]
);

firstMillion.connect();
```
```
Observer #2:208683
Observer #1:208683
Observer #2:432509
Observer #1:432509
Observer #2:644270
Observer #1:644270
Observer #2:887885
Observer #1:887885
Sequence #2 complete
Sequence #1 complete
```

## ConnectableObservable.connect( )
#### instructs a Connectable Observable to begin emitting items
Call a Connectable Observable's `connect( )` method to instruct it to begin emitting the items from its underlying Observable to its Observers.

The `connect( )` method returns a `Subscription`. You can call that object's `unsubscribe( )` method to instruct the Observable to stop emitting items to its Observers.

You can also use the `connect( )` method to instruct an Observable to begin emitting items (or, to begin generating items that would be emitted) even before any Observer has subscribed to it.

## Observable.publish( ) and Observable.multicast( )
#### represents an Observable as a Connectable Observable
To represent an Observable as a Connectable Observable, use the `publish( )` or `multicast()` method.

## Observable.replay( )
#### ensures that all Observers see the same sequence of emitted items, even if they subscribe after the Observable begins emitting items
[[images/rx-operators/replay.png]]