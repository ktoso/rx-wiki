* `aggregate( )` — _see `reduce( )`_
* [**`all( )`**](Conditional-and-Boolean-Operators#wiki-all) — determine whether all items emitted by an Observable meet some criteria
* [**`amb( )`**](Conditional-and-Boolean-Operators#wiki-amb) — given two or more source Observables, emits all of the items from the first of these Observables to emit an item
* (`rxjava-joins`) [**`and( )`**](Combining-Observables#wiki-and-then-and-when) — combine the emissions from two or more source Observables into a `Pattern`
* (`rxjava-async`) [**`asyncAction( )`**](Async-Operators#wiki-toasync-or-asyncaction-or-asyncfunc) — convert an Action into an Observable that executes the Action and emits its return value
* (`rxjava-async`) [**`asyncFunc( )`**](Async-Operators#wiki-toasync-or-asyncaction-or-asyncfunc) — convert a function into an Observable that executes the function and emits its return value
* (`rxjava-math`) [**`averageDouble( )`**](Mathematical-and-Aggregate-Operators#wiki-averageinteger-averagelong-averagefloat-and-averagedouble) — calculates the average of Doubles emitted by an Observable and emits this average
* (`rxjava-math`) [**`averageFloat( )`**](Mathematical-and-Aggregate-Operators#wiki-averageinteger-averagelong-averagefloat-and-averagedouble) — calculates the average of Floats emitted by an Observable and emits this average
* (`rxjava-math`) [**`averageInteger( )`**](Mathematical-and-Aggregate-Operators#wiki-averageinteger-averagelong-averagefloat-and-averagedouble) — calculates the average of Integers emitted by an Observable and emits this average
* (`rxjava-math`) [**`averageLong( )`**](Mathematical-and-Aggregate-Operators#wiki-averageinteger-averagelong-averagefloat-and-averagedouble) — calculates the average of Longs emitted by an Observable and emits this average
* [**`buffer( )`**](Transforming-Observables#wiki-buffer) — periodically gather items from an Observable into bundles and emit these bundles rather than emitting the items one at a time
* [**`cache( )`**](Observable-Utility-Operators#wiki-cache) — remember the sequence of items emitted by the Observable and emit the same sequence to future Subscribers
* [**`cast( )`**](Transforming-Observables#wiki-cast) — cast all items from the source Observable into a particular type before reemitting them
* (⁇) [**`chunkify( )`**](Phantom-Operators#wiki-chunkify) — returns an iterable that periodically returns a list of items emitted by the source Observable since the last list
* [**`combineLatest( )`**](Combining-Observables#wiki-combinelatest) — when an item is emitted by either of two Observables, combine the latest item emitted by each Observable via a specified function and emit items based on the results of this function
* [**`concat( )`**](Mathematical-and-Aggregate-Operators#wiki-concat) — concatenate two or more Observables sequentially
* [**`connect( )`**](Connectable-Observable-Operators#wiki-connectableobservableconnect) — instructs a Connectable Observable to begin emitting items
* [**`contains( )`**](Conditional-and-Boolean-Operators#wiki-contains) — determine whether an Observable emits a particular item or not
* [**`count( )`**](Mathematical-and-Aggregate-Operators#wiki-count-and-longcount) — counts the number of items emitted by an Observable and emits this count
* [**`create( )`**](Creating-Observables#wiki-create) — create an Observable from scratch by means of a function
* [**`debounce( )`**](Filtering-Observables#wiki-throttlewithtimeout-or-debounce) — only emit an item from the source Observable after a particular timespan has passed without the Observable emitting any other items
* [**`defaultIfEmpty( )`**](Conditional-and-Boolean-Operators#wiki-defaultifempty) — emit items from the source Observable, or emit a default item if the source Observable completes after emitting no items
* [**`defer( )`**](Creating-Observables#wiki-defer) — do not create the Observable until a Subscriber subscribes; create a fresh Observable on each subscription
* (`rxjava-async`) [**`deferFuture( )`**](Async-Operators#wiki-deferfuture) — convert a Future that returns an Observable into an Observable, but do not attempt to get the Observable that the Future returns until a Subscriber subscribes
* (⁇)(`rxjava-async`) [**`deferCancellableFuture( )`**](Phantom-Operators#wiki-fromcancellablefuture-startcancellablefuture-and-defercancellablefuture-) — convert a Future that returns an Observable into an Observable in a way that monitors the subscription status of the Observable to determine whether to halt work on the Future, but do not attempt to get the returned Observable until a Subscriber subscribes
* [**`delay( )`**](Observable-Utility-Operators#wiki-delay) — shift the emissions from an Observable forward in time by a specified amount
* [**`dematerialize( )`**](Observable-Utility-Operators#wiki-dematerialize) — convert a materialized Observable back into its non-materialized form
* [**`distinct( )`**](Filtering-Observables#wiki-distinct) — suppress duplicate items emitted by the source Observable
* [**`distinctUntilChanged( )`**](Filtering-Observables#wiki-distinctuntilchanged) — suppress duplicate consecutive items emitted by the source Observable
* [**`doOnCompleted( )`**](Observable-Utility-Operators#wiki-dooncompleted) — register an action to take when an Observable completes successfully
* [**`doOnEach( )`**](Observable-Utility-Operators#wiki-dooneach) — register an action to take whenever an Observable emits an item
* [**`doOnError( )`**](Observable-Utility-Operators#wiki-doonerror) — register an action to take when an Observable completes with an error
* [**`doOnTerminate( )`**](Observable-Utility-Operators#wiki-doonterminate) — register an action to take when an Observable completes, either successfully or with an error
* (`contrib-computation-expressions`) [**`doWhile( )`**](Conditional-and-Boolean-Operators#wiki-dowhile) — emit the source Observable's sequence, and then repeat the sequence as long as a condition remains true
* [**`elementAt( )`**](Filtering-Observables#wiki-elementat) — emit item _n_ emitted by the source Observable
* [**`elementAtOrDefault( )`**](Filtering-Observables#wiki-elementatordefault) — emit item _n_ emitted by the source Observable, or a default item if the source Observable emits fewer than _n_ items
* [**`empty( )`**](Creating-Observables#wiki-empty-error-and-never) — create an Observable that emits nothing and then completes
* [**`error( )`**](Creating-Observables#wiki-empty-error-and-never) — create an Observable that emits nothing and then signals an error
* [**`exists( )`**](Conditional-and-Boolean-Operators#wiki-exists-and-isempty) — determine whether an Observable emits any items or not
* [**`filter( )`**](Filtering-Observables#wiki-filter) — filter items emitted by an Observable
* [**`finallyDo( )`**](Observable-Utility-Operators#wiki-finallydo) — register an action to take when an Observable completes
* [**`first( )`**](Filtering-Observables#wiki-first-and-takefirst) (`Observable`) — emit only the first item emitted by an Observable, or the first item that meets some condition
* [**`first( )`**](Blocking-Observable-Operators#wiki-first-and-firstordefault) (`BlockingObservable`) — emit only the first item emitted by an Observable, or the first item that meets some condition
* [**`firstOrDefault( )`**](Filtering-Observables#wiki-firstordefault) (`Observable`) — emit only the first item emitted by an Observable, or the first item that meets some condition, or a default value if the source Observable is empty
* [**`firstOrDefault( )`**](Blocking-Observable-Operators#wiki-first-and-firstordefault) (`BlockingObservable`) — emit only the first item emitted by an Observable, or the first item that meets some condition, or a default value if the source Observable is empty
* [**`flatMap( )`**](Transforming-Observables#wiki-flatmap) — transform the items emitted by an Observable into Observables, then flatten this into a single Observable
* [**`forEach( )`**](Blocking-Observable-Operators#wiki-foreach) — invoke a function on each item emitted by the Observable; block until the Observable completes
* (`rxjava-async`) [**`forEachFuture( )`**](Async-Operators#wiki-foreachfuture) (`Async`) — pass Subscriber methods to an Observable but also have it behave like a Future that blocks until it completes
* (⁇) [**`forEachFuture( )`**](Phantom-Operators#wiki-foreachfuture) (`BlockingObservable`)— create a futureTask that will invoke a specified function on each item emitted by an Observable 
* (⁇) [**`forIterable( )`**](Phantom-Operators#wiki-foriterable) — apply a function to the elements of an Iterable to create Observables which are then concatenated
* [**`from( )`**](Creating-Observables#wiki-from) — convert an Iterable, a Future, or an Array into an Observable
* (`rxjava-async`) [**`fromAction( )`**](Async-Operators#wiki-fromaction) — convert an Action into an Observable that invokes the action and emits its result when a Subscriber subscribes
* (`rxjava-async`) [**`fromCallable( )`**](Async-Operators#wiki-fromcallable) — convert a Callable into an Observable that invokes the callable and emits its result or exception when a Subscriber subscribes
* (⁇)(`rxjava-async`) [**`fromCancellableFuture( )`**](Phantom-Operators#wiki-fromcancellablefuture-startcancellablefuture-and-defercancellablefuture-) — convert a Future into an Observable in a way that monitors the subscription status of the Observable to determine whether to halt work on the Future, but do not attempt to get the Future's value until a Subscriber subscribes
* (`rxjava-async`) `fromFunc0( )` — _see `fromCallable( )`_
* (⁇) [**`fromFuture( )`**](Phantom-Operators#wiki-fromfuture) — convert a Future into an Observable, but do not attempt to get the Future's value until a Subscriber subscribes
* (`rxjava-async`) [**`fromRunnable( )`**](Async-Operators#wiki-fromrunnable) — convert a Runnable into an Observable that invokes the runable and emits its result when a Subscriber subscribes
* (⁇) [**`generate( )`**](Phantom-Operators#wiki-generate-and-generateabsolutetime) — create an Observable that emits a sequence of items as generated by a function of your choosing
* (⁇) [**`generateAbsoluteTime( )`**](Phantom-Operators#wiki-generate-and-generateabsolutetime) — create an Observable that emits a sequence of items as generated by a function of your choosing, with each item emitted at an item-specific time
* [**`getIterator( )`**](Blocking-Observable-Operators#wiki-transformations-tofuture-toiterable-and-toiteratorgetiterator) — convert the sequence emitted by the Observable into an Iterator
* [**`groupBy( )`**](Transforming-Observables#wiki-groupby-and-groupbyuntil) — divide an Observable into a set of Observables that emit groups of items from the original Observable, organized by key
* [**`groupByUntil( )`**](Transforming-Observables#wiki-groupby-and-groupbyuntil) — divide an Observable into a set of Observables that emit groups of items from the original Observable, organized by key, and opening a new set periodically
* [**`groupJoin( )`**](Combining-Observables#wiki-join-and-groupjoin) — combine the items emitted by two Observables whenever one item from one Observable falls within a window of duration specified by an item emitted by the other Observable
* (`contrib-computation-expressions`) [**`ifThen( )`**](Conditional-and-Boolean-Operators#wiki-ifthen) — only emit the source Observable's sequence if a condition is true, otherwise emit an empty or default sequence
* [**`ignoreElements( )`**](Filtering-Observables#wiki-ignoreelements) — discard the items emitted by the source Observable and only pass through the error or completed notification
* [**`interval( )`**](Creating-Observables#wiki-interval) — create an Observable that emits a sequence of integers spaced by a given time interval
* [**`isEmpty( )`**](Conditional-and-Boolean-Operators#wiki-exists-and-isempty) — determine whether an Observable emits any items or not
* [**`join( )`**](Combining-Observables#wiki-join-and-groupjoin) — combine the items emitted by two Observables whenever one item from one Observable falls within a window of duration specified by an item emitted by the other Observable
* [**`just( )`**](Creating-Observables#wiki-just) — convert an object into an Observable that emits that object
* [**`last( )`**](Blocking-Observable-Operators#wiki-last-and-lastordefault) (`BlockingObservable`) — block until the Observable completes, then return the last item emitted by the Observable
* [**`last( )`**](Filtering-Observables#wiki-last) (`Observable`) — emit only the last item emitted by the source Observable
* [**`lastOrDefault( )`**](Blocking-Observable-Operators#wiki-last-and-lastordefault) (`BlockingObservable`) — block until the Observable completes, then return the last item emitted by the Observable or a default item if there is no last item
* [**`lastOrDefault( )`**](Filtering-Observables#wiki-lastOrDefault) (`Observable`) — emit only the last item emitted by an Observable, or a default value if the source Observable is empty
* [**`latest( )`**](Blocking-Observable-Operators#wiki-latest) — returns an iterable that blocks until or unless the Observable emits an item that has not been returned by the iterable, then returns the latest such item
* [**`longCount( )`**](Mathematical-and-Aggregate-Operators#wiki-count-and-longcount) — counts the number of items emitted by an Observable and emits this count
* [**`map( )`**](Transforming-Observables#wiki-map) — transform the items emitted by an Observable by applying a function to each of them
* **`mapMany( )`** — _see: `flatMap( )`_
* [**`materialize( )`**](Observable-Utility-Operators#wiki-materialize) — convert an Observable into a list of Notifications
* (`rxjava-math`) [**`max( )`**](Mathematical-and-Aggregate-Operators#wiki-max) — emits the maximum value emitted by a source Observable
* (`rxjava-math`) [**`maxBy( )`**](Mathematical-and-Aggregate-Operators#wiki-maxby) — emits the item emitted by the source Observable that has the maximum key value
* [**`merge( )`**](Combining-Observables#wiki-merge) — combine multiple Observables into one
* [**`mergeDelayError( )`**](Combining-Observables#wiki-mergedelayerror) — combine multiple Observables into one, allowing error-free Observables to continue before propagating errors
* (`rxjava-math`) [**`min( )`**](Mathematical-and-Aggregate-Operators#wiki-min) — emits the minimum value emitted by a source Observable
* (`rxjava-math`) [**`minBy( )`**](Mathematical-and-Aggregate-Operators#wiki-minby) — emits the item emitted by the source Observable that has the minimum key value
* [**`mostRecent( )`**](Blocking-Observable-Operators#wiki-mostrecent) — returns an iterable that always returns the item most recently emitted by the Observable
* [**`multicast( )`**](Connectable-Observable-Operators#wiki-observablepublish-and-observablemulticast) — represents an Observable as a Connectable Observable
* [**`never( )`**](Creating-Observables#wiki-empty-error-and-never) — create an Observable that emits nothing at all
* [**`next( )`**](Blocking-Observable-Operators#wiki-next) — returns an iterable that blocks until the Observable emits another item, then returns that item
* [**`observeOn( )`**](Observable-Utility-Operators#wiki-observeon) — specify on which Scheduler a Subscriber should observe the Observable
* [**`ofType( )`**](Filtering-Observables#wiki-oftype) — emit only those items from the source Observable that are of a particular class
* [**`onErrorFlatMap( )`**](Error-Handling-Operators#wiki-onerrorflatmap) — instructs an Observable to emit a sequence of items whenever it encounters an error
* [**`onErrorResumeNext( )`**](Error-Handling-Operators#wiki-onerrorresumenext) — instructs an Observable to emit a sequence of items if it encounters an error
* [**`onErrorReturn( )`**](Error-Handling-Operators#wiki-onerrorreturn) — instructs an Observable to emit a particular item when it encounters an error
* [**`onExceptionResumeNext( )`**](Error-Handling-Operators#wiki-onexceptionresumenext) — instructs an Observable to continue emitting items after it encounters an exception (but not another variety of throwable)
* [**`parallel( )`**](Observable-Utility-Operators#wiki-parallel) — split the work done on the emissions from an Observable into multiple Observables each operating on its own parallel thread
* [**`parallelMerge( )`**](Combining-Observables#wiki-parallelmerge) — combine multiple Observables into smaller number of Observables
* [**`pivot( )`**](Transforming-Observables#wiki-pivot) — combine multiple sets of grouped observables so that they are arranged primarily by group rather than by set
* [**`publish( )`**](Connectable-Observable-Operators#wiki-observablepublish-and-observablemulticast) — represents an Observable as a Connectable Observable
* [**`publishLast( )`**](Connectable-Observable-Operators#wiki-observablepublishlast) — represent an Observable as a Connectable Observable that emits only the last item emitted by the source Observable
* [**`range( )`**](Creating-Observables#wiki-range) — create an Observable that emits a range of sequential integers
* [**`reduce( )`**](Mathematical-and-Aggregate-Operators#wiki-reduce) — apply a function to each emitted item, sequentially, and emit only the final accumulated value
* [**`refCount( )`**](Connectable-Observable-Operators#wiki-connectableobservablerefcount) — makes a Connectable Observable behave like an ordinary Observable
* [**`repeat( )`**](Creating-Observables#wiki-repeat) — create an Observable that emits a particular item or sequence of items repeatedly
* [**`replay( )`**](Connectable-Observable-Operators#wiki-observablereplay) — ensures that all Subscribers see the same sequence of emitted items, even if they subscribe after the Observable begins emitting the items
* [**`retry( )`**](Error-Handling-Operators#wiki-retry) — if a source Observable emits an error, resubscribe to it in the hopes that it will complete without error
* [**`sample( )`**](Filtering-Observables#wiki-sample-or-throttlelast) — emit the most recent items emitted by an Observable within periodic time intervals
* [**`scan( )`**](Transforming-Observables#wiki-scan) — apply a function to each item emitted by an Observable, sequentially, and emit each successive value
* [**`sequenceEqual( )`**](Conditional-and-Boolean-Operators#wiki-sequenceequal) — test the equality of sequences emitted by two Observables
* [**`serialize( )`**](Observable-Utility-Operators#wiki-serialize) — force an Observable to make synchronous calls and to be well-behaved
* [**`single( )`**](Blocking-Observable-Operators#wiki-single-and-singleordefault) (`BlockingObservable`) — if the source Observable completes after emitting a single item, return that item, otherwise throw an exception
* [**`single( )`**](Observable-Utility-Operators#wiki-single-and-singleordefault) (`Observable`) — if the source Observable completes after emitting a single item, emit that item, otherwise notify of an exception
* [**`singleOrDefault( )`**](Blocking-Observable-Operators#wiki-single-and-singleordefault) (`BlockingObservable`) — if the source Observable completes after emitting a single item, return that item, otherwise return a default item
* [**`singleOrDefault( )`**](Observable-Utility-Operators#wiki-single-and-singleordefault) (`Observable`) — if the source Observable completes after emitting a single item, emit that item, otherwise emit a default item
* [**`skip( )`**](Filtering-Observables#wiki-skip) — ignore the first _n_ items emitted by an Observable
* [**`skipLast( )`**](Filtering-Observables#wiki-skiplast) — ignore the last _n_ items emitted by an Observable
* [**`skipUntil( )`**](Conditional-and-Boolean-Operators#wiki-skipuntil) — discard items emitted by a source Observable until a second Observable emits an item, then emit the remainder of the source Observable's items
* [**`skipWhile( )`**](Conditional-and-Boolean-Operators#wiki-skipwhile-and-skipwhilewithindex) — discard items emitted by an Observable until a specified condition is false, then emit the remainder
* [**`skipWhileWithIndex( )`**](Conditional-and-Boolean-Operators#wiki-skipwhile-and-skipwhilewithindex) — discard items emitted by an Observable until a specified condition is false, then emit the remainder
* (`rxjava-async`) [**`start( )`**](Async-Operators#wiki-start) — create an Observable that emits the return value of a function
* (⁇)(`rxjava-async`) [**`startCancellableFuture( )`**](Phantom-Operators#wiki-fromcancellablefuture-startcancellablefuture-and-defercancellablefuture-) — convert a function that returns Future into an Observable that emits that Future's return value in a way that monitors the subscription status of the Observable to determine whether to halt work on the Future
* (`rxjava-async`) [**`startFuture( )`**](Async-Operators#wiki-startfuture) — convert a function that returns Future into an Observable that emits that Future's return value
* [**`startWith( )`**](Combining-Observables#wiki-startwith) — emit a specified sequence of items before beginning to emit the items from the Observable
* [**`subscribeOn( )`**](Observable-Utility-Operators#wiki-subscribeon) — specify which Scheduler an Observable should use when its subscription is invoked
* (`rxjava-math`) [**`sumDouble( )`**](Mathematical-and-Aggregate-Operators#wiki-suminteger-sumlong-sumfloat-and-sumdouble) — adds the Doubles emitted by an Observable and emits this sum
* (`rxjava-math`) [**`sumFloat( )`**](Mathematical-and-Aggregate-Operators#wiki-suminteger-sumlong-sumfloat-and-sumdouble) — adds the Floats emitted by an Observable and emits this sum
* (`rxjava-math`) [**`sumInteger( )`**](Mathematical-and-Aggregate-Operators#wiki-suminteger-sumlong-sumfloat-and-sumdouble) — adds the Integers emitted by an Observable and emits this sum
* (`rxjava-math`) [**`sumLong( )`**](Mathematical-and-Aggregate-Operators#wiki-suminteger-sumlong-sumfloat-and-sumdouble) — adds the Longs emitted by an Observable and emits this sum
* (`contrib-computation-expressions`) [**`switchCase( )`**](Conditional-and-Boolean-Operators#wiki-switchcase) — emit the sequence from a particular Observable based on the results of an evaluation
* [**`switchOnNext( )`**](Combining-Observables#wiki-switchonnext) — convert an Observable that emits Observables into a single Observable that emits the items emitted by the most-recently emitted of those Observables
* `synchronize( )` — _see `serialize( )`_
* [**`take( )`**](Filtering-Observables#wiki-take) — emit only the first _n_ items emitted by an Observable
* [**`takeFirst( )`**](Filtering-Observables#wiki-first-and-takefirst) — emit only the first item emitted by an Observable, or the first item that meets some condition
* [**`takeLast( )`**](Filtering-Observables#wiki-takelast) — only emit the last _n_ items emitted by an Observable
* [**`takeLastBuffer( )`**](Filtering-Observables#wiki-takelastbuffer) — emit the last _n_ items emitted by an Observable, as a single list item
* [**`takeUntil( )`**](Conditional-and-Boolean-Operators#wiki-takeuntil) — emits the items from the source Observable until a second Observable emits an item
* [**`takeWhile( )`**](Conditional-and-Boolean-Operators#wiki-takewhile-and-takewhilewithindex) — emit items emitted by an Observable as long as a specified condition is true, then skip the remainder
* [**`takeWhileWithIndex( )`**](Conditional-and-Boolean-Operators#wiki-takewhile-and-takewhilewithindex) — emit items emitted by an Observable as long as a specified condition is true, then skip the remainder
* (`rxjava-joins`) [**`then( )`**](Combining-Observables#wiki-and-then-and-when) — transform a series of `Pattern` objects via a `Plan` template
* [**`throttleFirst( )`**](Filtering-Observables#wiki-throttlefirst) — emit the first items emitted by an Observable within periodic time intervals
* [**`throttleLast( )`**](Filtering-Observables#wiki-sample-or-throttlelast) — emit the most recent items emitted by an Observable within periodic time intervals
* [**`throttleWithTimeout( )`**](Filtering-Observables#wiki-throttlewithtimeout-or-debounce) — only emit an item from the source Observable after a particular timespan has passed without the Observable emitting any other items
* [**`timeInterval( )`**](Observable-Utility-Operators#wiki-timeinterval) — emit the time lapsed between consecutive emissions of a source Observable
* [**`timeout( )`**](Filtering-Observables#wiki-timeout) — emit items from a source Observable, but issue an exception if no item is emitted in a specified timespan
* [**`timer( )`**](Creating-Observables#wiki-timer) — create an Observable that emits a single item after a given delay
* [**`timestamp( )`**](Observable-Utility-Operators#wiki-timestamp) — attach a timestamp to every item emitted by an Observable
* (`rxjava-async`) [**`toAsync( )`**](Async-Operators#wiki-toasync-or-asyncaction-or-asyncfunc) — convert a function or Action into an Observable that executes the function and emits its return value
* [**`toBlockingObservable( )`**](Blocking-Observable-Operators) — transform an Observable into a BlockingObservable
* [**`toFuture( )`**](Blocking-Observable-Operators#wiki-transformations-tofuture-toiterable-and-toiteratorgetiterator) — convert the Observable into a Future
* [**`toIterable( )`**](Blocking-Observable-Operators#wiki-transformations-tofuture-toiterable-and-toiteratorgetiterator) — convert the sequence emitted by the Observable into an Iterable
* [**`toIterator( )`**](Blocking-Observable-Operators#wiki-transformations-tofuture-toiterable-and-toiteratorgetiterator) — convert the sequence emitted by the Observable into an Iterator
* [**`toList( )`**](Mathematical-and-Aggregate-Operators#wiki-tolist) — collect all items from an Observable and emit them as a single List
* [**`toMap( )`**](Mathematical-and-Aggregate-Operators#wiki-tomap-and-tomultimap) — convert the sequence of items emitted by an Observable into a map keyed by a specified key function
* [**`toMultimap( )`**](Mathematical-and-Aggregate-Operators#wiki-tomap-and-tomultimap) — convert the sequence of items emitted by an Observable into an ArrayList that is also a map keyed by a specified key function
* [**`toSortedList( )`**](Mathematical-and-Aggregate-Operators#wiki-tosortedlist) — collect all items from an Observable and emit them as a single, sorted List
* [**`using( )`**](Observable-Utility-Operators#wiki-using) — create a disposable resource that has the same lifespan as an Observable
* (`rxjava-joins`) [**`when( )`**](Combining-Observables#wiki-and-then-and-when) — convert a series of `Plan` objects into an Observable
* **`where( )`** — _see: `filter( )`_
* (`contrib-computation-expressions`) [**`whileDo( )`**](Conditional-and-Boolean-Operators#wiki-whiledo) — if a condition is true, emit the source Observable's sequence and then repeat the sequence as long as the condition remains true
* [**`window( )`**](Transforming-Observables#wiki-window) — periodically subdivide items from an Observable into Observable windows and emit these windows rather than emitting the items one at a time 
* [**`zip( )`**](Combining-Observables#wiki-zip) — combine sets of items emitted by two or more Observables together via a specified function and emit items based on the results of this function

(⁇) — this proposed operator is not part of RxJava as of 0.18.0