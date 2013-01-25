<a name='Hello-World'/>
# Hello World!

A requisite "Hello World!" which creates an Observable from a list of Strings, subscribes to the Observable with a function that will print "Hello %!" for each string.

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
=> (hello ["Ben" "George"])
Hello Ben!
Hello George!
```
