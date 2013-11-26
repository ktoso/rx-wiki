This section explains the [`ConnectableObservable`](http://netflix.github.io/RxJava/javadoc/rx/observables/ConnectableObservable.html) subclass and its operators:

* [**`ConnectableObservable.connect( )`**](Connectable-Observable-Operators#connectableobservableconnect) — instructs a Connectable Observable to begin emitting items
* [**`Observable.publish( )` and `Observable.multicast( )`**](Connectable-Observable-Operators#observablepublish-and-observablemulticast) — represents an Observable as a Connectable Observable
* [**`Observable.publishLast( )`**](Connectable-Observable-Operators#observablepublishlast) — represent an Observable as a Connectable Observable that emits only the last item emitted by the source Observable
* [**`Observable.replay( )`**](Connectable-Observable-Operators#observablereplay) — ensures that all Observers see the same sequence of emitted items, even if they subscribe after the Observable begins emitting the items
* [**`ConnectableObservable.refCount( )`**](Connectable-Observable-Operators#connectableobservablerefcount) — makes a Connectable Observable behave like an ordinary Observable

A Connectable Observable resembles an ordinary Observable, except that it does not begin emitting items when it is subscribed to, but only when its `connect()` method is called. In this way you can wait for all intended Observers to subscribe to the Observable before the Observable begins emitting items.

[[images/rx-operators/publishConnect.png]]

The following example code shows two Observers subscribing to the same Observable. In the first case, they subscribe to an ordinary Observable; in the second case, they subscribe to a Connectable Observable that only connects after both observers subscribe. Note the difference in the output:

**Example #1:**
```groovy
def firstMillion  = Observable.range( 1, 1000000 ).sample(7, java.util.concurrent.TimeUnit.MILLISECONDS);

firstMillion.subscribe(
  { println("Observer #1:" + it); },    // onNext
  { println("Error encountered"); },    // onError
  { println("Sequence #1 complete"); }  // onCompleted
);

firstMillion.subscribe(
  { println("Observer #2:" + it); },    // onNext
  { println("Error encountered"); },    // onError
  { println("Sequence #2 complete"); }  // onCompleted
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
  { println("Observer #1:" + it); },    // onNext
  { println("Error encountered"); },    // onError
  { println("Sequence #1 complete"); }  // onCompleted
);

firstMillion.subscribe(
  { println("Observer #2:" + it); },    // onNext
  { println("Error encountered"); },    // onError
  { println("Sequence #2 complete"); }  // onCompleted
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

#### see also:
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/observables/ConnectableObservable.html">`ConnectableObservable`</a>
* <a href="http://www.introtorx.com/Content/v1.0.10621.0/14_HotAndColdObservables.html#PublishAndConnect">Introduction to Rx: Publish and Connect</a>

***

## ConnectableObservable.connect( )
#### instructs a Connectable Observable to begin emitting items
Call a Connectable Observable's `connect( )` method to instruct it to begin emitting the items from its underlying Observable to its Observers.

The `connect( )` method returns a `Subscription`. You can call that object's `unsubscribe( )` method to instruct the Observable to stop emitting items to its Observers.

You can also use the `connect( )` method to instruct an Observable to begin emitting items (or, to begin generating items that would be emitted) even before any Observer has subscribed to it.

#### see also
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/observables/ConnectableObservable.html#connect()">`connect()`</a>
* RxJS: <a href="https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md#connectableobservableprototypeconnect">`connect`</a>
* Linq: <a href="http://msdn.microsoft.com/en-us/library/hh211748.aspx">`Connect`</a>
* <a href="http://www.introtorx.com/Content/v1.0.10621.0/14_HotAndColdObservables.html#PublishAndConnect">Introduction to Rx: Publish and Connect</a>

***

## Observable.publish( ) and Observable.multicast( )
#### represents an Observable as a Connectable Observable
To represent an Observable as a Connectable Observable, use the `publish( )` or `multicast()` method.

#### see also:
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#multicast(rx.subjects.Subject)">`multicast(subject)`</a>
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#publish()">`publish()`</a>
* RxJS: <a href="https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md#rxobservableprototypemulticastsubject--subjectselector-selector">`multicast`</a> and <a href="https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md#rxobservableprototypepublishselector">`publish`</a>
* Linq: <a href="http://msdn.microsoft.com/en-us/library/system.reactive.linq.observable.multicast.aspx">`Multicast`</a> and <a href="http://msdn.microsoft.com/en-us/library/system.reactive.linq.observable.publish.aspx">`Publish`</a>
* <a href="http://www.introtorx.com/Content/v1.0.10621.0/14_HotAndColdObservables.html#PublishAndConnect">Introduction to Rx: Publish and Connect</a>
* <a href="http://www.introtorx.com/Content/v1.0.10621.0/14_HotAndColdObservables.html#Multicast">Introduction to Rx: Multicast</a>

***

## Observable.publishLast( )
#### represent an Observable as a Connectable Observable that emits only the last item emitted by the source Observable
[[images/rx-operators/publishLast.png]]

#### see also:
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#publishLast()">`publishLast()`</a>
* RxJS: <a href="https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md#rxobservableprototypepublishlatestselector">`publishLast`</a>
* Linq: <a href="http://msdn.microsoft.com/en-us/library/system.reactive.linq.observable.publishlast.aspx">`PublishLast`</a>
* <a href="http://www.introtorx.com/Content/v1.0.10621.0/14_HotAndColdObservables.html#PublishLast">Introduction to Rx: PublishLast</a>

***

## Observable.replay( )
#### ensures that all Observers see the same sequence of emitted items, even if they subscribe after the Observable begins emitting items
[[images/rx-operators/replay.png]]

#### see also:
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#replay()">`replay()`</a>
* RxJS: <a href="https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md#rxobservableprototypereplayselector-buffersize-window-scheduler">`replay`</a>
* Linq: <a href="http://msdn.microsoft.com/en-us/library/system.reactive.linq.observable.replay.aspx">`Replay`</a>
* <a href="http://www.introtorx.com/Content/v1.0.10621.0/14_HotAndColdObservables.html#Replay">Introduction to Rx: Replay</a>

***

## ConnectableObservable.refCount( )
#### makes a Connectable Observable behave like an ordinary Observable
[[images/rx-operators/publishRefCount.png]]

You can represent a Connectable Observable so that it behaves much like an ordinary Observable by using the `refCount( )` operator. This operator keeps track of how many Observers are subscribed to the resulting Observable and refrains from disconnecting from the source ConnectableObservable until all such Observables unsubscribe.

#### see also:
* RxJS: <a href="https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md#connectableobservableprototyperefcount">`refCount`</a>
* Linq: <a href="http://msdn.microsoft.com/en-us/library/hh211664.aspx">`RefCount`</a>
* <a href="http://www.introtorx.com/Content/v1.0.10621.0/14_HotAndColdObservables.html#RefCount">Introduction to Rx: RefCount</a>