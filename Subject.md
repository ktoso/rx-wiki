A <a href="http://netflix.github.io/RxJava/javadoc/rx/subjects/Subject.html">``Subject``</a> is a sort of bridge or proxy that acts both as an ``Observer`` and as an ``Observable``. Because it is an Observer, it can subscribe to one or more Observables, and because it is an Observable, it can pass through the items it observes by reemitting them, and it can also emit new items.

#### see also:
* <a href="http://www.introtorx.com/Content/v1.0.10621.0/02_KeyTypes.html#Subject">Introduction to Rx: Subject</a>

There are four subclasses of ``Subject`` that are designed for particular use cases:
# AsyncSubject
<a href="http://netflix.github.io/RxJava/javadoc/rx/subjects/AsyncSubject.html">``AsyncSubject``</a> emits the last value (and only the last value) emitted by the source Observable(s), and only after that source Observable(s) completes. (If the source Observable does not emit any values, the ``AsyncSubject`` also completes without emitting any values.)
[[images/rx-operators/S.AsyncSubject.png]]

#### see also:
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/subjects/AsyncSubject.html">`AsyncSubject`</a>
* Reactive Extensions: <a href="http://msdn.microsoft.com/en-us/library/hh229363(v=vs.103).aspx">`AsyncSubject`</a>
* <a href="http://www.introtorx.com/Content/v1.0.10621.0/02_KeyTypes.html#AsyncSubject">Introduction to Rx: AsyncSubject</a>

# BehaviorSubject
When an Observer subscribes to a <a href="http://netflix.github.io/RxJava/javadoc/rx/subjects/BehaviorSubject.html">``BehaviorSubject``</a>, it begins by emitting the item most recently emitted by the source Observable (or a seed/default value if none has yet been emitted) and then continues to emit any other items emitted later by the source Observable(s).
[[images/rx-operators/S.BehaviorSubject.png]]

#### see also:
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/subjects/BehaviorSubject.html">`BehaviorSubject`</a>
* Reactive Extensions: <a href="http://msdn.microsoft.com/en-us/library/hh211949(v=vs.103).aspx">`BehaviorSubject`</a>
* <a href="http://www.introtorx.com/Content/v1.0.10621.0/02_KeyTypes.html#BehaviorSubject">Introduction to Rx: BehaviorSubject</a>

# PublishSubject
<a href="http://netflix.github.io/RxJava/javadoc/rx/subjects/PublishSubject.html">``PublishSubject``</a> emits to a subscriber only those items that are emitted by the source Observable(s) subsequent to the time of the subscription.
[[images/rx-operators/S.PublishSubject.png]]

#### see also:
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/subjects/PublishSubject.html">`PublishSubject`</a>


# ReplaySubject
<a href="http://netflix.github.io/RxJava/javadoc/rx/subjects/ReplaySubject.html">``ReplaySubject``</a> emits to any subscriber all of the items that were emitted by the source Observable(s), regardless of when the subscriber subscribes.
[[images/rx-operators/S.ReplaySubject.png]]

#### see also:
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/subjects/ReplaySubject.html">`ReplaySubject`</a>
* Reactive Extensions: <a href="http://msdn.microsoft.com/en-us/library/hh211810(v=vs.103).aspx">`ReplaySubject`</a>
* <a href="http://www.introtorx.com/Content/v1.0.10621.0/02_KeyTypes.html#ReplaySubject">Introduction to Rx: ReplaySubject</a>