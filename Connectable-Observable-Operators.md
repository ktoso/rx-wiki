This section explains the `ConnectableObservable` subclass and its operators:

* **`ConnectableObservable.connect( )`** — instructs a Connectable Observable to begin emitting values
* **`Observable.publish( )`** — represents an Observable as a Connectable Observable
* **`Observable.multicast( )`** — 
* **`Observable.replay( )`** — ensures that all observers see the same sequence, even if they subscribe after the Observable begins emitting the sequence

A Connectable Observable resembles an ordinary Observable, except that it does not begin emitting a sequence of values when it is subscribed to, but only when its `connect()` method is called. In this way you can wait for all intended observers to subscribe to the Observable before the Observable begins emitting values.

[[images/rx-operators/publishConnect.png]]

The following example code shows two observers subscribing to the same Observable. In the first case, they subscribe to an ordinary Observable; in the second case, they subscribe to a Connectable Observable that only connects after both observers subscribe. Note the difference in the output:

**Example #1:**
```groovy
def firstMillion  = Observable.range( 1, 1000000 ).sample(7, java.util.concurrent.TimeUnit.MILLISECONDS);

firstMillion.subscribe(
  [ onNext:{ myWriter.println("Subscriber #1:" + it); },
    onCompleted:{ myWriter.println("Sequence #1 complete"); },
    onError:{ myWriter.println("Error encountered"); } ]
);

firstMillion.subscribe(
  [ onNext:{ myWriter.println("Subscriber #2:" + it); },
    onCompleted:{ myWriter.println("Sequence #2 complete"); },
    onError:{ myWriter.println("Error encountered"); } ]
);
```
```
Subscriber #1:211128
Subscriber #1:411633
Subscriber #1:629605
Subscriber #1:841903
Sequence #1 complete
Subscriber #2:244776
Subscriber #2:431416
Subscriber #2:621647
Subscriber #2:826996
Sequence #2 complete
```
**Example #2:**
```groovy
def firstMillion  = Observable.range( 1, 1000000 ).sample(7, java.util.concurrent.TimeUnit.MILLISECONDS).publish();

firstMillion.subscribe(
  [ onNext:{ myWriter.println("Subscriber #1:" + it); },
    onCompleted:{ myWriter.println("Sequence #1 complete"); },
    onError:{ myWriter.println("Error encountered"); } ]
);

firstMillion.subscribe(
  [ onNext:{ myWriter.println("Subscriber #2:" + it); },
    onCompleted:{ myWriter.println("Sequence #2 complete"); },
    onError:{ myWriter.println("Error encountered"); } ]
);

firstMillion.connect();
```
```
Subscriber #2:208683
Subscriber #1:208683
Subscriber #2:432509
Subscriber #1:432509
Subscriber #2:644270
Subscriber #1:644270
Subscriber #2:887885
Subscriber #1:887885
Sequence #2 complete
Sequence #1 complete
```

## ConnectableObservable.connect( )
#### instructs a Connectable Observable to begin emitting values
Call a Connectable Observable's `connect( )` method to instruct it to begin emitting the objects from its underlying Observable to its subscribing observers.

The `connect( )` method returns a `Subscription`. You can call that object's `unsubscribe( )` method to instruct the Observable to stop emitting values to its subscribers.

You can also use the `connect( )` method to instruct an Observable to begin emitting values (or, to begin generating values that would be emitted anyway) even before any observer has subscribed to it.

## Observable.publish( )
#### represents an Observable as a Connectable Observable
To represent an Observable as a Connectable Observable, use the Observable's `publish( )` method.

## Observable.multicast( )
#### 

## Observable.replay( )
#### ensures that all observers see the same sequence, even if they subscribe after the Observable begins emitting the sequence

[[images/rx-operators/replay.png]]