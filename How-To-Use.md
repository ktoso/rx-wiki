<a name='Hello-World'/>
# Hello World!

A requisite "Hello World!" which creates an Observable from a list of Strings, subscribes to the Observable with a function that will print "Hello %!" for each string.

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