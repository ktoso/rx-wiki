<a name='Hello-World'/>
# Hello World!

Requisite first example which creates an Observable from a list of Strings, subscribes to the Observable with a function that will print "Hello [arg]!" for each string.

> This example is given first in Java and then other languages to provide comparison.

> Subsequent examples will use a mixture of languages all of which can be found in the /src/examples folders of each [language adaptor](https://github.com/Netflix/RxJava/tree/master/language-adaptors).

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

This first examples uses Groovy to create an Observable that emits a sequence of 75 strings.

It is purposefully written verbosely with static typing and implementation of the Func1 anonymous inner class for clarity around what's happening:

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

Here is the same code in Clojure that uses a Future (instead of raw thread) and implemented more consisely:

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

Results

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

Note that all of the above examples ignore error handling for brevity, see below for examples including error handling.

More information can be found on the [[Observable]] and [[Creation Operators|Observable-Operators-Creation]] pages.

# Composition

Rx allows chaining operators together to transform and compose sequences.

Using Groovy this first example uses a previously defined asynchronous sequence that emits 75 elements, skips the first 10 then takes the next 5 and transforms them before subscribing and printing the values:

```groovy
/**
 * Asynchronously calls 'customObservableNonBlocking' and defines
 * a chain of operators to apply to the callback sequence.
 */
def simpleComposition() {
    customObservableNonBlocking().skip(10).take(5)
        .map({ stringValue -> return stringValue + "_transformed"})
        .subscribe({ println "onNext => " + it})
}
```

This results in:

```text
onNext => anotherValue_10_transformed
onNext => anotherValue_11_transformed
onNext => anotherValue_12_transformed
onNext => anotherValue_13_transformed
onNext => anotherValue_14_transformed
```

This next example in Clojure consumes 3 asynchronous Observable sequences including a dependency from one to another and composes them into a single response object:

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
          ; now combine 3 async sequences using zip
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

# Error Handling

Here is a revised version of the Wikipedia example above with error handling:

```groovy
/**
 * Fetch a list of Wikipedia articles asynchronously with error handling.
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
            } catch(Exception e) {
                observer.onError(e);
            }
        }
            return Observable.noOpSubscription();
    });
}
```

Notice how it now calls `onError(Exception e)` if an exception occurs and in the following it passes a second closure that handles onError.

```groovy
fetchWikipediaArticleAsynchronouslyWithErrorHandling("Tiger", "NonExistentTitle", "Elephant")
    .subscribe(
        { println "--- Article ---\n" + it.substring(0, 125)}, 
        { println "--- Error ---\n" + it.getMessage()})
```

See more information on [[Error Handling|ErrorHandling-Operators]] including functions such as `onErrorResumeNext()` which allows providing default sequences to continue with in event of error.