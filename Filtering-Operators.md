This section explains operators for filtering and selecting elements.


## filter()

#### Filter elements from an Observable sequence

[[images/operation-filter.png]]

You can filter a Observable, discarding any values that do not meet some test, by passing a filtering closure into the `Observable.filter()` method. For example, the following code filters a list of integers, emitting only those that are even (that is, where the remainder from dividing the number by two is zero):

```groovy
numbers = Observable.toObservable([1, 2, 3, 4, 5, 6, 7, 8, 9]);

Observable.filter(numbers, { 0 == (it % 2) }).subscribe(
  [ onNext:{ response.getWriter().println(it); },
    onCompleted:{ response.getWriter().println("Sequence complete"); },
    onError:{ response.getWriter().println("Error encountered"); } ]
);

2
4
6
8
Sequence complete
```

In addition to calling `filter()` as a stand-alone method, you can also call it as a method of a Observable object, so, in the example above, instead of 

```groovy
Observable.filter(numbers, { 0 == (it %2) }) ...
```
you could instead write 

```groovy
numbers.filter({ 0 == (it % 2) }) ...
```



## last()

#### Only emit the last element

[[images/operation-last.png]]

To convert a Observable that emits several objects into one that only emits the last of these objects before completing, use the `last()` method. For instance, in the following code, `last()` emits only the last integer in the list of integers represented by `numbers`:

```groovy
numbers = Observable.toObservable([1, 2, 3, 4, 5, 6, 7, 8, 9]);

Observable.last(numbers).subscribe(
  [ onNext:{ response.getWriter().println(it); },
    onCompleted:{ response.getWriter().println("Sequence complete"); },
    onError:{ response.getWriter().println("Error encountered"); } ]
);

9
Sequence complete
```

In addition to calling `last()` as a stand-alone method, you can also call it as a method of a Observable object, so, in the example above, instead of 

```groovy
Observable.last(numbers) ...
``` 
you could instead write

```groovy
numbers.last() ...
```