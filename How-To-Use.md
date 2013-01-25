# Hello World!


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