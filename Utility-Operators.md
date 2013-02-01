This section explains various utility operators for working with Observables.


## materialize()

#### Convert an Observable into a list of Notifications

[[images/operation-materialize.png]]

A well-formed Observable will call its observer’s `onNext` closure zero or more times, and then will call either the `onCompleted` or `onError` closure exactly once. The `Observable.materialize()` method converts this series of calls into a series of emissions from a Observable, where it represents each such call as a `Notification` object.

For example:

```groovy
numbers = Observable.toObservable([1, 2, 3]);

Observable.materialize(numbers).subscribe(
  [ onNext: { if(Kind.OnNext == it.kind) response.getWriter().println("Next: " + it.value);
              else if(Kind.OnCompleted == it.kind) response.getWriter().println("Completed");
              else if(Kind.OnError == it.kind) response.getWriter().println("Error: " + it.exception); },
    onCompleted:{ response.getWriter().println("Sequence complete"); },
    onError:{ response.getWriter().println("Error encountered"); } ]
);

Next: 1
Next: 2
Next: 3
Completed
Sequence complete
```

In addition to calling `materialize()` as a stand-alone method, you can also call it as a method of a Observable object, so that instead of 

```groovy
Observable.materialize(numbers) ...
```
in the above example, you could also write 

```groovy
numbers.materialize() ...
```