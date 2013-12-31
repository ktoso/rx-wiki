The operators described on this page are part of the `Async` class and are used to convert synchronous methods into Observables.

* [**`start( )`**](Async-Operators#start) — create an Observable that emits the return value of a function
* [**`toAsync( )` or `asyncAction( )` or `asyncFunc( )`**](Async-Operators#toasync-or-asyncaction-or-asyncfunc) — convert a function or Action into an Observable that executes the function and emits its return value
* [**`startFuture( )`**](Async-Operators#startfuture) — convert a function that returns Future into an Observable that emits that Future's return value
* [**`deferFuture( )`**](Async-Operators#deferfuture) — convert a Future that returns an Observable into an Observable, but do not attempt to get the Observable that the Future returns until an Observer subscribes
* [**`fromCancellableFuture( )`, `startCancellableFuture( )`, and `deferCancellableFuture( )`**](Async-Operators#fromcancellablefuture-startcancellablefuture-and-defercancellablefuture-) — versions of Future-to-Observable converters that monitor the subscription status of the Observable to determine whether to halt work on the Future
* [**`forEachFuture( )`**](Async-Operators#foreachfuture) — pass observer methods to an Observable but also have it behave like a Future that blocks until it completes
* [**`fromAction( )`**](Async-Operators#fromaction) — convert an Action into an Observable that invokes the action and emits its result when an observer subscribes
* [**`fromFunc0( )`**](Async-Operators#fromfunc0) — convert a Func0 into an Observable that invokes the function and emits its result when an observer subscribes
* [**`fromCallable( )`**](Async-Operators#fromcallable) — convert a Callable into an Observable that invokes the callable and emits its result or exception when an observer subscribes
* [**`fromRunnable( )`**](Async-Operators#fromrunnable) — convert a Runnable into an Observable that invokes the runable and emits its result when an observer subscribes

***

## start( )
#### create an Observable that emits the return value of a function
[[images/rx-operators/start.png]]

Pass the `start( )` method a function that returns a value, and `start( )` will execute that function asynchronously and return an Observable that will emit that value to any subsequent Observers.

#### see also:
* RxJS: <a href="https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md#rxobservablestartfunc-scheduler-context">`start`</a>
* Linq: <a href="http://msdn.microsoft.com/en-us/library/hh229265.aspx">`Start`</a>

***

## toAsync( ) or asyncAction( ) or asyncFunc( )
#### convert a function into an Observable that executes the function and emits its return value
[[images/rx-operators/toAsync.png]]

With `toAsync( )` you can create an Observable that, when it is subscribed to, executes a function (or Action) of your choosing and emits its return value before completing. In the case of an `Action`, it will emit `null` before completing. Note that even if the resulting Observable is subscribed to more than once, the function will only be executed once, and its sole return value will be emitted to all future observers.

#### see also:
* Linq: <a href="http://msdn.microsoft.com/en-us/library/system.reactive.linq.observable.toasync.aspx">`ToAsync`</a>
* RxJS: <a href="https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md#rxobservabletoasyncfunc-scheduler-context">`toAsync`</a>

***

## startFuture( )
#### convert a function that returns Future into an Observable that emits that Future's return value
[[images/rx-operators/startFuture.png]]

The `startFuture( )` method is similar to the `fromFuture( )` method except that it calls the function to obtain the Future immediately, and attempts to get its value even before an Observer subscribes to the resulting Observable. It then holds this value and returns it to any future observer.

***

## deferFuture( )
#### convert a Future that returns an Observable into an Observable, but do not attempt to get the Observable that the Future returns until an Observer subscribes
[[images/rx-operators/deferFuture.png]]

You can use the `deferFuture( )` operator to convert a Future that returns an Observable into an Observable that emits the values of that returned Observable in such a way that the Future is not invoked until an observer subscribes to the resulting Observable.

***

## fromCancellableFuture( ), startCancellableFuture( ), and deferCancellableFuture( )
#### versions of Future-to-Observable converters that monitor the subscription status of the Observable to determine whether to halt work on the Future

If the a subscriber to the Observable that results when a Future is converted to an Observable later unsubscribes from that Observable, it can be useful to have the ability to stop attempting to retrieve items from the Future. The "cancellable" Future enables you do do this. These three methods will return Observables that, when unsubscribed to, will also "unsubscribe" from the underlying Futures.

***

## forEachFuture( )
#### pass observer methods to an Observable but also have it behave like a Future that blocks until it completes
[[images/rx-operators/forEachFuture.png]]

You pass `forEachFuture( )` some subset of observer methods (`onNext`, `onError`, and `onCompleted`) and the Observable will call them in the usual way. But `forEachFuture( )` itself returns a `Future` that blocks until the source Observable completes, and then returns either the completion or the error, depending on how the Observable completed.

***

# fromAction( )
#### convert an Action into an Observable that invokes the action and emits its result when an observer subscribes
[[images/rx-operators/fromAction.png]]

***

# fromFunc0( )
#### convert a Func0 into an Observable that invokes the function and emits its result when an observer subscribes
[[images/rx-operators/fromFunc0.png]]

***

# fromCallable( )
#### convert a Callable into an Observable that invokes the callable and emits its result or exception when an observer subscribes
[[images/rx-operators/fromCallable.png]]

***

# fromRunnable( )
#### convert a Runnable into an Observable that invokes the runable and emits its result when an observer subscribes
[[images/rx-operators/fromRunnable.png]]
