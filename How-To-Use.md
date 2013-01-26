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

### asdf


More information can be found on the [[Observable]] and [[Creation Operators|Observable-Operators-Creation]] pages.

# Composition

# Error Handling

OnError subscribe, resume next, delay