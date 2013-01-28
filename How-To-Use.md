<a name='Hello-World'/>
# Hello World!

Requisite first example which creates an Observable from a list of Strings, subscribes to the Observable with a function that will print "Hello [arg]!" for each string.

> This example is given first in Java and then other languages to provide comparison.

> Subsequent examples will use a mixture of languages all of which can be found in the [rxjava-examples](https://github.com/Netflix/RxJava/tree/master/rxjava-examples) submodule.

### Java

```java
public static void hello(String... names) {
    Observable.toObservable(names).subscribe(new Action1<String>() {

        @Override
        public void call(String s) {
            System.out.println("Hello " + s + "!");
        }

    });
}
```

```java
hello("Ben", "George");
Hello Ben!
Hello George!
```

### Groovy

```groovy
def hello(String[] names) {
    Observable.toObservable(names)
        .subscribe({ println "Hello " + it + "!"})
}
```

```groovy
hello("Ben", "George")
Hello Ben!
Hello George!
```

### Clojure

```clojure
(defn hello
  [&rest]
  (-> (Observable/toObservable &rest)
    (.subscribe #(println (str "Hello " % "!")))))
```

```
(hello ["Ben" "George"])
Hello Ben!
Hello George!
```

# Creating Observable Sequences

An Observable sequence originates from two sources, an existing data structure or an Observable implementation which synchronously or asynchronously executes and passes data via `onNext()`.

## Existing Data

The Observable `toObservable`, `from` and `just` methods allow converting any object, list or array of objects into an observable sequence:

```groovy
Observable<Integer> o = Observable.toObservable(1, 2, 3, 4, 5, 6);

Observable<String> o = Observable.from("a", "b", "c");

def list = [5, 6, 7, 8]
Observable<Integer> o = Observable.toObservable(list);

Observable<String> o = Observable.just("one object");
```

These sequences will synchronously invoke `onNext()` on an Observer when subscribed to for each object and then call `onCompleted()`.

## Observable Implementation

Asynchronous IO or computational operations or "infinite" streams of data can be implemented using the Observable class.

This can be done either by extending the Observable class or by using the `Observable.create()` factory method. 

### Synchronous Observable

```groovy
/**
 * This example shows a custom Observable that blocks 
 * when subscribed to (does not spawn an extra thread).
 * 
 * @return Observable<String>
 */
def customObservableBlocking() {
    return Observable.create(new Func1<Observer<String>, Subscription>() {
        def Subscription call(Observer<String> observer) {
            for(int i=0; i<50; i++) {
                observer.onNext("value_" + i);
            }
            // after sending all values we complete the sequence
            observer.onCompleted();
            // return a NoOpSubsription since this blocks and thus
            // can't be unsubscribed from
            return Observable.noOpSubscription();
        };
    });
}

// To see output:
customObservableBlocking().subscribe({ println(it)});
```

### Asynchronous Observable

```groovy
/**
 * This example shows a custom Observable that does not block
 * when subscribed to as it spawns a separate thread.
 *
 * @return Observable<String>
 */
def customObservableNonBlocking() {
    return Observable.create(new Func1<Observer<String>, Subscription>() {
        /**
         * This 'call' method will be invoked with the Observable is subscribed to.
         * 
         * It spawns a thread to do it asynchronously.
         */
        def Subscription call(Observer<String> observer) {
            // For simplicity this example uses a Thread instead of an ExecutorService/ThreadPool
            final Thread t = new Thread(new Runnable() {
                void run() {
                    for(int i=0; i<75; i++) {
                        observer.onNext("anotherValue_" + i);
                    }
                    // after sending all values we complete the sequence
                    observer.onCompleted();
                };
            });
            t.start();
        
            return new Subscription() {
                public void unsubscribe() {
                    // Ask the thread to stop doing work.
                    // For this simple example it just interrupts.
                    t.interrupt();
                }
            };
        };
    });
}

// To see output:
customObservableNonBlocking().subscribe({ println(it)});
```

Here is another example in Clojure that uses a Future and executes on a thread-pool:

```clojure
(defn customObservableNonBlocking []
  "This example shows a custom Observable that does not block 
   when subscribed to as it spawns a separate thread.
   
  returns Observable<String>"
  (Observable/create 
    (fn [observer]
      (let [f (future 
                (doseq [x (range 50)] (-> observer (.onNext (str "anotherValue_" x))))
                ; after sending all values we complete the sequence
                (-> observer .onCompleted))
            ; a subscription that cancels the future if unsubscribed
            subscription (Observable/createSubscription #(-> f (.cancel true)))]
        ))
      ))
```

```clojure
; To see output
(.subscribe (customObservableNonBlocking) #(println %))
```

Here is an example that fetches articles from Wikipedia and calls onNext with each one:

```clojure
(defn fetchWikipediaArticleAsynchronously [wikipediaArticleNames]
  "Fetch a list of Wikipedia articles asynchronously.
  
   return Observable<String> of HTML"
  (Observable/create 
    (fn [observer]
      (let [f (future
                (doseq [articleName wikipediaArticleNames]
                  (-> observer (.onNext (http/get (str "http://en.wikipedia.org/wiki/" articleName)))))
                ; after sending response to onnext we complete the sequence
                (-> observer .onCompleted))
            ; a subscription that cancels the future if unsubscribed
            subscription (Observable/createSubscription #(-> f (.cancel true)))]
        ))
      ))
```

```clojure
(-> (fetchWikipediaArticleAsynchronously ["Tiger" "Elephant"]) 
  (.subscribe #(println "--- Article ---\n" (subs (:body %) 0 125) "...")))
```


More information can be found on the [[Observable]] and [[Creation Operators|Observable-Operators-Creation]] pages.

# Composition

# Error Handling

OnError subscribe, resume next, delay