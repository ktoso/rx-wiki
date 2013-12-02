This section explains operators that handle errors and exceptions encountered by Observables.

* [**`onErrorResumeNext( )`**](Error-Handling-Operators#onerrorresumenext) — instructs an Observable to continue emitting items after it encounters an error
* [**`onErrorReturn( )`**](Error-Handling-Operators#onerrorreturn) — instructs an Observable to emit a particular item when it encounters an error
* [**`onExceptionResumeNextViaObservable( )`**](Error-Handling-Operators#onexceptionresumenextviaobservable) — instructs an Observable to continue emitting items after it encounters an exception (but not another variety of throwable)
* [**`retry( )`**](Error-Handling-Operators#retry) — if a source Observable emits an error, resubscribe to it in the hopes that it will complete without error

***

## onErrorResumeNext( )
#### instructs an Observable to attempt to continue emitting items after it encounters an error
[[images/rx-operators/onErrorResumeNext.png]]

The `onErrorResumeNext( )` method returns an Observable that mirrors the behavior of the source Observable, _unless_ that Observable invokes `onError( )` in which case, rather than propagating that error to the Observer, `onErrorResumeNext( )` will instead begin mirroring a second, backup Observable, as shown in the following sample code:
```groovy
def myObservable = Observable.create({ anObserver ->
  anObserver.onNext('Three');
  anObserver.onNext('Two');
  anObserver.onNext('One');
  anObserver.onError();
});
def myFallback = Observable.create({ anObserver ->
  anObserver.onNext('0');
  anObserver.onNext('1');
  anObserver.onNext('2');
  anObserver.onCompleted();
});

myObservable.onErrorResumeNext(myFallback).subscribe(
  { println(it); },                          // onNext
  { println("Error: " + it.getMessage()); }, // onError
  { println("Sequence complete"); }          // onCompleted
);```
```
Three
Two
One
0
1
2
Sequence complete
```

#### see also:
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#onErrorResumeNext(rx.util.functions.Func1)">`onErrorResumeNext(throwable,function)`</a>
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#onErrorResumeNext(rx.Observable)">`onErrorResumeNext(sequence)`</a>
* RxJS: <a href="https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md#rxobservableprototypeonerrorresumenextsecond">`onErrorResumeNext`</a>
* Linq: <a href="http://msdn.microsoft.com/en-us/library/system.reactive.linq.observable.onerrorresumenext.aspx">`OnErrorResumeNext`</a>
* <a href="http://www.introtorx.com/Content/v1.0.10621.0/11_AdvancedErrorHandling.html#OnErrorResumeNext">Introduction to Rx: OnErrorResumeNext</a>

***

## onErrorReturn( )
#### instructs an Observable to emit a particular item to an Observer’s onNext method when it encounters an error
[[images/rx-operators/onErrorReturn.png]]

The `onErrorReturn( )` method returns an Observable that mirrors the behavior of the source Observable, _unless_ that Observable invokes `onError( )` in which case, rather than propagating that error to the Observer, `onErrorReturn( )` will instead emit a specified item and invoke the Observer's `onCompleted( )` method, as shown in the following sample code:
```groovy
def myObservable = Observable.create({ anObserver ->
  anObserver.onNext('Four');
  anObserver.onNext('Three');
  anObserver.onNext('Two');
  anObserver.onNext('One');
  anObserver.onError();
});

myObservable.onErrorReturn({ return('Blastoff!'); }).subscribe(
  { println(it); },                          // onNext
  { println("Error: " + it.getMessage()); }, // onError
  { println("Sequence complete"); }          // onCompleted
);
```
```
Four
Three
Two
One
Blastoff!
Sequence complete
```

#### see also:
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#onErrorReturn(rx.util.functions.Func1)">`onErrorReturn(func)`</a>

***

## onExceptionResumeNextViaObservable( )
#### instructs an Observable to continue emitting items after it encounters an exception (but not another variety of throwable)
[[images/rx-operators/onExceptionResumeNextViaObservable.png]]

Much like `onErrorResumeNext( )` method, this returns an Observable that mirrors the behavior of the source Observable, _unless_ that Observable invokes `onError( )` in which case, if the `Throwable` passed to `onError( )` is an `Exception`, rather than propagating that `Exception` to the Observer, `onExceptionResumeNextViaObservable( )` will instead begin mirroring a second, backup Observable. If the `Throwable` is not an `Exception`, the Observable returned by `onExceptionResumeNextViaObservable( )` will propagate it to its observers' `onError( )` method and will not invoke its backup Observable.

#### see also:
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#onExceptionResumeNext(rx.Observable)">`onExceptionResumeNext(observable)`</a>

***

## retry( )
#### if a source Observable emits an error, resubscribe to it in the hopes that it will complete without error
[[images/rx-operators/retry.png]]

The `retry( )` method responds to an `onError( )` call from the source Observable by not passing that call through to its observers, but instead resubscribing to the source Observable and giving it another opportunity to complete its sequence without error. You can pass `retry( )` a maximum number of retry-attempts, or you can pass nothing, in which case it will never stop retrying to get an error-free sequence. It always passes `onNext( )` calls through to its observers, even from sequences that terminate with an error, so this can cause duplicate emissions (as shown in the diagram above).

#### see also:
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#retry()">`retry()`</a>
* javadoc: <a href="http://netflix.github.io/RxJava/javadoc/rx/Observable.html#retry(int)">`retry(count)`</a>
* RxJS: <a href="https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md#rxobservableprototyperetryretrycount">`retry`</a>
* Linq: <a href="http://msdn.microsoft.com/en-us/library/system.reactive.linq.observable.retry.aspx">`Retry`</a>
* <a href="http://www.introtorx.com/Content/v1.0.10621.0/11_AdvancedErrorHandling.html#Retry">Introduction to Rx: Retry</a>