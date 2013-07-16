RxJava is an implementation of [Reactive Extensions](https://rx.codeplex.com) – a library for composing asynchronous and event-based programs that use observable sequences – for the Java VM.

It extends the [observer pattern](http://en.wikipedia.org/wiki/Observer_pattern) to support sequences of data/events and adds operators that compose sequences together declaratively while abstracting away low-level threading, synchronization, thread-safety, concurrent data structures, non-blocking IO, and other such concerns.

It supports Java 5 or higher and JVM based languages such as [Groovy](https://github.com/Netflix/RxJava/tree/master/language-adaptors/rxjava-groovy), [Clojure](https://github.com/Netflix/RxJava/tree/master/language-adaptors/rxjava-clojure), [Scala](https://github.com/Netflix/RxJava/tree/master/language-adaptors/rxjava-scala) and [JRuby](https://github.com/Netflix/RxJava/tree/master/language-adaptors/rxjava-jruby).

# Why?

### Futures are Expensive to Compose

<a href="http://docs.oracle.com/javase/7/docs/api/java/util/concurrent/Future.html">Java Futures</a> are straightforward to use for a <a href="https://gist.github.com/4670979">single level of asynchronous execution</a> but they start to add <a href="https://gist.github.com/4671081">non-trivial complexity</a> when they’re nested.

It is <a href="https://gist.github.com/4671081#file-futuresb-java-L163">difficult to use Futures to optimally compose conditional asynchronous execution flows</a> (or impossible, as latencies of each request vary at runtime). It <a href="http://www.amazon.com/gp/product/0321349601?ie=UTF8&tag=none0b69&linkCode=as2&camp=1789&creative=9325&creativeASIN=0321349601">can be done</a> of course, but it quickly becomes complicated (and thus error prone) or prematurely blocks on “Future.get()” - eliminating the benefit of asynchronous execution.

### Futures are Less-Flexible in terms of the Data They Work With

RxJava’s Observables support not just the emission of single scalar values (as Futures do), but also of sequences of values or even infinite streams. Observable is a single abstraction that can be used for any of these use cases. An Observable has all of the flexibility and elegance associated with its mirror-image cousin the Iterable.

### RxJava is More Lightweight, Less Restrictive than Akka Futures

The RxJava implementation is not biased toward some particular source of concurrency or asynchronicity. It also tries to be very lightweight (a single JAR focused on just the Observable abstraction and related higher-order functions).

A composable Future could be implemented just as generically, but <a href="http://doc.akka.io/docs/akka/2.2.0/java.html">Akka Futures</a> for example come tied in with an Actor library and a lot of other stuff. RxJava tries not to restrict you in this way. You can choose to implement your Observables using actors, thread-pools, event loops, non-blocking I/O, or whatever implementation suits your needs, your style, or your expertise.

### Callbacks Have Their Own Problems

Callbacks offer a solution to the tendency to block on Future.get() by not allowing anything to block. They are naturally efficient because they execute when the response is ready.

But as with Futures, while callbacks are easy to use with a single level of asynchronous execution, <a href="https://gist.github.com/4677544">they become unwieldy with nested composition</a>.

### RxJava is a Polyglot Implementation

RxJava is meant for a more polyglot environment than just Java/Scala, and it is being designed to respect the idioms of each language. (<a href="https://github.com/Netflix/RxJava/pull/304">This is something we’re still working on.</a>)

# Functional Reactive Programming (FRP)

RxJava offers efficient execution and composition by providing a collection of operators with which you can filter, select, transform, combine, and compose Observables.

The Observable class can be thought of as a “push” equivalent to <a href="http://docs.oracle.com/javase/7/docs/api/java/lang/Iterable.html">Iterable</a>, which is “pull.” With an Iterable, the consumer pulls values from the producer and the thread blocks until those values arrive. By contrast, with an Observable the producer pushes values to the consumer whenever values are available. This approach is more flexible, because values can arrive synchronously or asynchronously.

The Observable type adds two missing semantics to the Gang of Four’s <a href="http://en.wikipedia.org/wiki/Observer_pattern">Observer pattern</a>, to match those that are available in the Iterable type:  

1. the ability for the producer to signal to the consumer that there is no more data available
1. the ability for the producer to signal to the consumer that an error has occurred

With these additions, RxJava unifies the Iterable and Observable types. The only difference between them is the direction in which the data flows. This is very important because now any operation you can perform on an Iterable, you can also perform on an Observable. Here is an example:

```groovy
/**
 * Asynchronously calls 'customObservableNonBlocking' and defines 
 * a chain of operators to apply to the callback sequence.
 */
def simpleComposition() {
  // fetch an asynchronous Observable<String> 
  // that emits 75 Strings of 'anotherValue_#'
  customObservableNonBlocking()
    // skip the first 10
    .skip(10)
    // take the next 5
    .take(5)
    // transform each String with the provided function
    .map({ stringValue -> return stringValue + "_transformed" })
    // subscribe to the sequence and print each transformed String
    .subscribe({ println "onNext => " + it })
}
 
// output
onNext => anotherValue_10_transformed
onNext => anotherValue_11_transformed
onNext => anotherValue_12_transformed
onNext => anotherValue_13_transformed
onNext => anotherValue_14_transformed
```

# More Information

* QCon London 2013 presentation: [Functional Reactive Programming in the Netflix API](http://www.infoq.com/presentations/netflix-functional-rx) and [interview](http://www.infoq.com/interviews/christensen-hystrix-rxjava)
* [Functional Reactive in the Netflix API with RxJava](http://techblog.netflix.com/2013/02/rxjava-netflix-api.html)
* [Optimizing the Netflix API](http://techblog.netflix.com/2013/01/optimizing-netflix-api.html)
* [Reactive Programming at Netflix](http://techblog.netflix.com/2013/01/reactive-programming-at-netflix.html)
* [rx.codeplex.com](https://rx.codeplex.com)
* [Rx Design Guidelines (PDF)](http://go.microsoft.com/fwlink/?LinkID=205219)
* [Channel 9 MSDN videos on Reactive Extensions](http://channel9.msdn.com/Tags/reactive+extensions)
* [Your Mouse is a Database](http://queue.acm.org/detail.cfm?id=2169076)
* [Beginner’s Guide to the Reactive Extensions](http://msdn.microsoft.com/en-us/data/gg577611)
* [Wikipedia: Reactive Programming](http://en.wikipedia.org/wiki/Reactive_programming)
* [Wikipedia: Functional Reactive Programming](http://en.wikipedia.org/wiki/Functional_reactive_programming)
* [Tutorial: Functional Programming in Javascript](http://jhusain.github.com/learnrx/index.html)
* [What is (functional) reactive programming?](http://stackoverflow.com/a/1030631/1946802)
* [Rx Is now Open Source](http://www.hanselman.com/blog/ReactiveExtensionsRxIsNowOpenSource.aspx)
* [What is FRP? - Elm Language](http://elm-lang.org/learn/What-is-FRP.elm)

# RxJava Libraries

The following external libraries can work with RxJava:

* [Camel RX](http://camel.apache.org/rx.html) provides an easy way to reuse any of the [Apache Camel components, protocols, transports and data formats](http://camel.apache.org/components.html) with the RxJava API
* [rxjava-http-tail](https://github.com/myfreeweb/rxjava-http-tail) allows you to follow logs over HTTP, like `tail -f`