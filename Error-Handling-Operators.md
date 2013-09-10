This section explains operators that handle errors and exceptions encountered by Observables.

* [**`onErrorResumeNext( )`**](Error-Handling-Operators#onerrorresumenext) — instructs an Observable to continue emitting items after it encounters an error
* [**`onErrorReturn( )`**](Error-Handling-Operators#onerrorreturn) — instructs an Observable to emit a particular item when it encounters an error
* [**`onExceptionResumeNextViaObservable( )`**](Error-Handling-Operators#onexceptionresumenextviaobservable) — instructs an Observable to continue emitting items after it encounters an exception (but not another variety of throwable)

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
  [ onNext:{ myWriter.println(it); },
    onCompleted:{ myWriter.println("Sequence complete"); },
    onError:{ myWriter.println("Error encountered"); } ]
);
```
```
Three
Two
One
0
1
2
Sequence complete
```

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
  [ onNext:{ myWriter.println(it); },
    onCompleted:{ myWriter.println("Sequence complete"); },
    onError:{ myWriter.println("Error encountered"); } ]
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

## onExceptionResumeNextViaObservable( )
#### instructs an Observable to continue emitting items after it encounters an exception (but not another variety of throwable)
[[images/rx-operators/onExceptionResumeNextViaObservable.png]]

Much like `onErrorResumeNext( )` method, this returns an Observable that mirrors the behavior of the source Observable, _unless_ that Observable invokes `onError( )` in which case, if the `Throwable` passed to `onError( )` is an `Exception`, rather than propagating that `Exception` to the Observer, `onExceptionResumeNextViaObservable( )` will instead begin mirroring a second, backup Observable. If the `Throwable` is not an `Exception`, the Observable returned by `onExceptionResumeNextViaObservable( )` will propagate it to its observers' `onError( )` method and will not invoke its backup Observable.