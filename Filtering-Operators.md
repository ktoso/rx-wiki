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




## `Observable.skip( )`

[[images/operation-skip.png]]

You can ignore the first *n* items emitted by a Observable and attend only to those items that come after, by modifying the Observable with the `Observable.skip( *n* )` method.

```groovy
numbers = Observable.toObservable( [1, 2, 3, 4, 5, 6, 7, 8, 9] );

Observable.skip(numbers, 3).subscribe(
  [ onNext:{ response.getWriter().println( it ); },
    onCompleted:{ response.getWriter().println( "Sequence complete" ); },
    onError:{ response.getWriter().println( "Error encountered" ); } ]
);

`4`
`5`
`6`
`7`
`8`
`9`
`Sequence complete{`}
```

In addition to calling skip( ) as a stand-alone method, you can also call it as a method of a Observable object, so, in the example above, instead of 

```groovy
Observable.skip( numbers, 3 )…
``` you could instead write 

```groovy
numbers.skip(3)…
```

## `Observable.take( )`

[[images/operation-take.png]]

You can choose to pay attention only to the first *n* values emitted by a Observable by calling its `take( *n* )` method. That method returns a Observable that will call a subscribing observer’s `onNext` closure a maximum of *n* times before calling `onCompleted`. For example,

```groovy
numbers = Observable.toObservable( [1, 2, 3, 4, 5, 6, 7, 8, 9] );

Observable.take(numbers, 3).subscribe(
  [ onNext:{ response.getWriter().println( it ); },
    onCompleted:{ response.getWriter().println( "Sequence complete" ); },
    onError:{ response.getWriter().println( "Error encountered" ); } ]
);

`1`
`2`
`3`
`Sequence complete`
```

In addition to calling `take( )` as a stand-alone method, you can also call it as a method of a Observable object, so, in the example above, instead of 

```groovy
Observable.take( numbers, 3 )…
```
you could instead write 
```groovy
numbers.take(3)…
```

If you call `take( *n* )` on a Observable, and that Observable emits _fewer_ than *n* items before completing, the new, `take`\-modified Observable will _not_ throw an error, but will merely emit this same fewer number of items before it completes.