The operators described on this page are part of the distinct `rxjava-async` module. They are used to convert synchronous methods into Observables.

* [**`start( )`**](Async-Operators#wiki-start) — create an Observable that emits the return value of a function
* [**`toAsync( )` or `asyncAction( )` or `asyncFunc( )`**](Async-Operators#wiki-toasync-or-asyncaction-or-asyncfunc) — convert a function or Action into an Observable that executes the function and emits its return value
* [**`startFuture( )`**](Async-Operators#wiki-startfuture) — convert a function that returns Future into an Observable that emits that Future's return value
* [**`deferFuture( )`**](Async-Operators#wiki-deferfuture) — convert a Future that returns an Observable into an Observable, but do not attempt to get the Observable that the Future returns until a Subscriber subscribes
* [**`forEachFuture( )`**](Async-Operators#wiki-foreachfuture) — pass Subscriber methods to an Observable but also have it behave like a Future that blocks until it completes
* [**`fromAction( )`**](Async-Operators#wiki-fromaction) — convert an Action into an Observable that invokes the action and emits its result when a Subscriber subscribes
* [**`fromCallable( )`**](Async-Operators#wiki-fromcallable) — convert a Callable into an Observable that invokes the callable and emits its result or exception when a Subscriber subscribes
* [**`fromRunnable( )`**](Async-Operators#wiki-fromrunnable) — convert a Runnable into an Observable that invokes the runable and emits its result when a Subscriber subscribes

***

## start( )
#### create an Observable that emits the return value of a function
[[images/rx-operators/start.png]]

Pass the `start( )` method a function that returns a value, and `start( )` will execute that function asynchronously and return an Observable that will emit that value to any subsequent Subscribers.

#### see also:
* RxJS: <a href="https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md#rxobservablestartfunc-scheduler-context">`start`</a>
* Linq: <a href="http://msdn.microsoft.com/en-us/library/hh229265.aspx">`Start`</a>

***

## toAsync( ) or asyncAction( ) or asyncFunc( )
#### convert a function into an Observable that executes the function and emits its return value
[[images/rx-operators/toAsync.png]]

With `toAsync( )` you can create an Observable that, when it is subscribed to, executes a function (or Action) of your choosing and emits its return value before completing. In the case of an `Action`, it will emit `null` before completing. Note that even if the resulting Observable is subscribed to more than once, the function will only be executed once, and its sole return value will be emitted to all future Subscribers.

#### see also:
* Linq: <a href="http://msdn.microsoft.com/en-us/library/system.reactive.linq.observable.toasync.aspx">`ToAsync`</a>
* RxJS: <a href="https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md#rxobservabletoasyncfunc-scheduler-context">`toAsync`</a>

***

## startFuture( )
#### convert a function that returns Future into an Observable that emits that Future's return value
[[images/rx-operators/startFuture.png]]

The `startFuture( )` method is similar to the `fromFuture( )` method except that it calls the function to obtain the Future immediately, and attempts to get its value even before a Subscriber subscribes to the resulting Observable. It then holds this value and returns it to any future Subscriber.

***

## deferFuture( )
#### convert a Future that returns an Observable into an Observable, but do not attempt to get the Observable that the Future returns until a Subscriber subscribes
[[images/rx-operators/deferFuture.png]]

You can use the `deferFuture( )` operator to convert a Future that returns an Observable into an Observable that emits the values of that returned Observable in such a way that the Future is not invoked until a Subscriber subscribes to the resulting Observable.

***

## forEachFuture( )
#### pass Subscriber methods to an Observable but also have it behave like a Future that blocks until it completes
[[images/rx-operators/forEachFuture.png]]

You pass `forEachFuture( )` some subset of the Subscriber methods (`onNext`, `onError`, and `onCompleted`) and the Observable will call them in the usual way. But `forEachFuture( )` itself returns a `Future` that blocks until the source Observable completes, and then returns either the completion or the error, depending on how the Observable completed.

***

# fromAction( )
#### convert an Action into an Observable that invokes the action and emits its result when a Subscriber subscribes
[[images/rx-operators/fromAction.png]]

***

# fromCallable( )
#### convert a Callable into an Observable that invokes the callable and emits its result or exception when a Subscriber subscribes
[[images/rx-operators/fromCallable.png]]

***

# fromRunnable( )
#### convert a Runnable into an Observable that invokes the runable and emits its result when a Subscriber subscribes
[[images/rx-operators/fromRunnable.png]]
