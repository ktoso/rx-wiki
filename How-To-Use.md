<a name='Hello-World'/>
# Hello World!

These example implementations of “Hello World” in Java, Groovy, and Clojure create an Observable from a list of Strings, and then subscribe to this Observable with a method that prints “Hello _String_!” for each string emitted by the Observable.

> Subsequent examples will use a mixture of languages all of which can be found in the `/src/examples` folders of each [language adaptor](https://github.com/Netflix/RxJava/tree/master/language-adaptors).

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
        .subscribe({ println "Hello " + it + "!" })
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

# Creating Observables

To create an Observable, you can either implement an Observable that (synchronously or asynchronously) executes and emits data by invoking an Observer's `onNext()` method, or you can convert an existing data structure into an Observable by using some Observable methods that are designed for this purpose.

## Creating Observables from Existing Data Structures

You use the Observable `toObservable()`, `from()`, and `just()` methods to convert objects, lists, or arrays of objects into Observables:

```groovy
Observable<Integer> o = Observable.toObservable(1, 2, 3, 4, 5, 6);

Observable<String> o = Observable.from("a", "b", "c");

def list = [5, 6, 7, 8]
Observable<Integer> o = Observable.toObservable(list);

Observable<String> o = Observable.just("one object");
```

These converted Observables will synchronously invoke the `onNext()` method of any Observer that subscribes to them, for each item emitted by the Observable, and will then invoke the Observer’s `onCompleted()` method.

## Implementing an Observable

You can implement asynchronous IO, computational operations, or “infinite” streams of data by using the Observable class.

You can do this either by extending the Observable class or by using the `Observable.create()` factory method. 

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

This first example uses Groovy to create an Observable that emits 75 strings.

It is written verbosely, with static typing and implementation of the `Func1` anonymous inner class, to make the example more clear:

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
                        observer.onNext("value_" + i);
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
customObservableNonBlocking().subscribe({ println(it) });
```

Here is the same code in Clojure that uses a Future (instead of raw thread) and is implemented more consisely:

```clojure
(defn customObservableNonBlocking []
  "This example shows a custom Observable that does not block 
   when subscribed to as it spawns a separate thread.
   
  returns Observable<String>"
  (Observable/create 
    (fn [observer]
      (let [f (future 
                (doseq [x (range 50)] (-> observer (.onNext (str "value_" x))))
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

Here is an example that fetches articles from Wikipedia and invokes onNext with each one:

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
        ))))
```

```clojure
(-> (fetchWikipediaArticleAsynchronously ["Tiger" "Elephant"]) 
  (.subscribe #(println "--- Article ---\n" (subs (:body %) 0 125) "...")))
```

Back to Groovy, the same Wikipedia functionality but using closures instead of anonymous inner classes:

```groovy
/**
 * Fetch a list of Wikipedia articles asynchronously.
 * 
 * @param wikipediaArticleName
 * @return Observable<String> of HTML
 */
def fetchWikipediaArticleAsynchronously(String... wikipediaArticleNames) {
    return Observable.create({ Observer<String> observer ->
        Thread.start {
            for(articleName in wikipediaArticleNames) {
                observer.onNext(new URL("http://en.wikipedia.org/wiki/"+articleName).getText());
            }
            observer.onCompleted();
        }
        return Observable.noOpSubscription();
    });
}

fetchWikipediaArticleAsynchronously("Tiger", "Elephant")
    .subscribe({ println "--- Article ---\n" + it.substring(0, 125)})
```

Results:

```text
--- Article ---
 <!DOCTYPE html>
<html lang="en" dir="ltr" class="client-nojs">
<head>
<title>Tiger - Wikipedia, the free encyclopedia</title> ...
--- Article ---
 <!DOCTYPE html>
<html lang="en" dir="ltr" class="client-nojs">
<head>
<title>Elephant - Wikipedia, the free encyclopedia</tit ...
```

Note that all of the above examples ignore error handling, for brevity. See below for examples that include error handling.

More information can be found on the [[Observable]] and [[Creation Operators|Creation-Operators]] pages.

# Composition

RxJava allows you to chain operators together to transform and compose Observables.

This first example, in Groovy, uses a previously defined, asynchronous Observable that emits 75 items, skips the first 10 of these, then takes the next 5 and transforms them before subscribing and printing the items:

```groovy
/**
 * Asynchronously calls 'customObservableNonBlocking' and defines
 * a chain of operators to apply to the callback sequence.
 */
def simpleComposition() {
    customObservableNonBlocking().skip(10).take(5)
        .map({ stringValue -> return stringValue + "_xform"})
        .subscribe({ println "onNext => " + it})
}
```

This results in:

```text
onNext => value_10_xform
onNext => value_11_xform
onNext => value_12_xform
onNext => value_13_xform
onNext => value_14_xform
```

Here is a marble diagram that illustrates this transformation:
[[images/rx-operators/Composition.1.png]]

This next example, in Clojure, consumes three asynchronous Observables, including a dependency from one to another, and emits a single response item:

```clojure
(defn getVideoForUser [userId videoId]
  "Get video metadata for a given userId
   - video metadata
   - video bookmark position
   - user data
  return Observable<Map>"
    (let [user-observable (-> (getUser userId)
              (.map (fn [user] {:user-name (:name user) :language (:preferred-language user)})))
          bookmark-observable (-> (getVideoBookmark userId videoId)
              (.map (fn [bookmark] {:viewed-position (:position bookmark)})))
          ; getVideoMetadata requires :language from user-observable so nest inside map function
          video-metadata-observable (-> user-observable 
              (.mapMany
                ; fetch metadata after a response from user-observable is received
                (fn [user-map] 
                  (getVideoMetadata videoId (:language user-map)))))]
          ; now combine 3 observables using zip
          (-> (Observable/zip bookmark-observable video-metadata-observable user-observable 
                (fn [bookmark-map metadata-map user-map]
                  {:bookmark-map bookmark-map 
                  :metadata-map metadata-map
                  :user-map user-map}))
            ; and transform into a single response object
            (.map (fn [data]
                  {:video-id videoId
                   :video-metadata (:metadata-map data)
                   :user-id userId
                   :language (:language (:user-map data))
                   :bookmark (:viewed-position (:bookmark-map data))
                  })))))
```

The response looks like this:

```clojure
{:video-id 78965, 
 :video-metadata {:video-id 78965, :title House of Cards: Episode 1, 
                  :director David Fincher, :duration 3365}, 
 :user-id 12345, :language es-us, :bookmark 0}
```

And here is a marble diagram that illustrates how that code produces that response:
[[images/rx-operators/Composition.2.png]]

The following example, in Groovy, comes from <a href="https://speakerdeck.com/benjchristensen/evolution-of-the-netflix-api-qcon-sf-2013">Ben Christensen's QCon presentation on the evolution of the Netflix API</a>:

```groovy
public Observable getVideoSummary(APIVideo video) {
   def seed = [id:video.id, title:video.getTitle();
   def bookmarkObservable = getBookmark(video);
   def artworkObservable = getArtworkImageUrl(video);
   return( Observable.merge(bookmarkObservable, artworkObservable)
      .reduce(seed, { aggregate, current -> aggregate << current })
      .map({ [(video.id.toString() : it] }))
}
```

And here is a marble diagram that illustrates how that code uses the `reduce` operator to bring the results from multiple Observables together in one structure:
[[images/rx-operators/Composition.3.png]]

# Error Handling

Here is a revised version of the Wikipedia example shown above, but with error handling:

```groovy
/**
 * Fetch a list of Wikipedia articles asynchronously, with error handling.
 *
 * @param wikipediaArticleName
 * @return Observable<String> of HTML
 */
def fetchWikipediaArticleAsynchronouslyWithErrorHandling(String... wikipediaArticleNames) {
    return Observable.create({ Observer<String> observer ->
        Thread.start {
            try {
                for(articleName in wikipediaArticleNames) {
                    observer.onNext(new URL("http://en.wikipedia.org/wiki/"+articleName).getText());
                }
                observer.onCompleted();
            } catch(Throwable t) {
                observer.onError(t);
            }
        }
            return Observable.noOpSubscription();
    });
}
```

Notice how it now invokes `onError(Throwable t)` if an error occurs and note that the following code passes `subscribe()` a second method that handles `onError`:

```groovy
fetchWikipediaArticleAsynchronouslyWithErrorHandling("Tiger", "NonExistentTitle", "Elephant")
    .subscribe(
        { println "--- Article ---\n" + it.substring(0, 125) }, 
        { println "--- Error ---\n" + it.getMessage() })
```

See the [Observable Utility Operators](https://github.com/Netflix/RxJava/wiki/Observable-Utility-Operators#onerrorresumenext) page for more information on specialized error handling techniques in RxJava, including methods like `onErrorResumeNext()` and `onErrorReturn()` that allow Observables to continue with fallbacks in the event of error.

Here is an example of how you can use such a method to pass along custom information about any exceptions you encounter. Imagine you have an Observable or cascade of Observables --- `myObservable` --- and you want to intercept any exceptions that would normally pass through to an Observer's `onError` method, replacing these with a customized Throwable of your own design. You could do this by modifying `myObservable` with the `onErrorResumeNext()` method, and passing into that method an Observable that calls `onError` with your customized Throwable (a utility method called `error()` will generate such an Observable for you):

```groovy
myModifiedObservable = myObservable.onErrorResumeNext({ t ->
   Throwable myThrowable = myCustomizedThrowableCreator(t);
   return(Observable.error(myThrowable));
});
```