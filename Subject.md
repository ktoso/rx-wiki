A <a href="http://netflix.github.io/RxJava/javadoc/rx/subjects/Subject.html">``Subject``</a> is a sort of bridge or proxy that acts both as an ``Observer`` and as an ``Observable``. Because it is an Observer, it can subscribe to one or more Observables, and because it is an Observable, it can pass through the items it observes by reemitting them, and it can also emit new items.

There are four subclasses of ``Subject`` that are designed for particular use cases:
# AsyncSubject
<a href="http://netflix.github.io/RxJava/javadoc/rx/subjects/AsyncSubject.html">``AsyncSubject``</a> emits the last value (and only the last value) emitted by the source Observable(s), and only after that source Observable(s) completes.
[[images/rx-operators/S.AsyncSubject.png]]

# BehaviorSubject
When an Observer subscribes to a <a href="http://netflix.github.io/RxJava/javadoc/rx/subjects/BehaviorSubject.html">``BehaviorSubject``</a>, it begins by emitting the item most recently emitted by the source Observable (or a seed/default value if none has yet been emitted) and then continues to emit any other items emitted later by the source Observable(s).
[[images/rx-operators/S.BehaviorSubject.png]]

# PublishSubject
<a href="http://netflix.github.io/RxJava/javadoc/rx/subjects/PublishSubject.html">``PublishSubject``</a> emits to a subscriber only those items that are emitted by the source Observable(s) subsequent to the time of the subscription.
[[images/rx-operators/S.PublishSubject.png]]

# ReplaySubject
<a href="http://netflix.github.io/RxJava/javadoc/rx/subjects/ReplaySubject.html">``ReplaySubject``</a> emits to any subscriber all of the items that were emitted by the source Observable(s), regardless of when the subscriber subscribes.
[[images/rx-operators/S.ReplaySubject.png]]
