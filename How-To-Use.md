<a name='Hello-World'/>
# Hello World!

A requisite "Hello World!" which creates an Observable from a list of Strings, subscribes to the Observable with a function that will print "Hello %!" for each string.

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

An observable sequence can 

  - from existing data
  - as an API for async
       - Give control of blocking/non-blocking to API
   - blocking
   - non-blocking

More information can be found on the [[Observer Pattern]] page.

# Composition

# Error Handling

OnError subscribe, resume next, delay