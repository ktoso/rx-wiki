* [**`aggregate( )`**](Transforming-Observables#reduce-or-aggregate) — apply a function to each emitted item, sequentially, and emit only the final accumulated value
* [**`all( )`**](Conditional-and-Boolean-Operators#all) — determine whether all items emitted by an Observable meet some criteria
* [**`amb( )`**](Conditional-and-Boolean-Operators#amb) — given two or more source Observables, emits all of the items from the first of these Observables to emit an item
* [**`and( )`**](Combining-Observables#and-then-and-when) — combine the emissions from two or more source Observables into a `Pattern`
* [**`average( )`**](Mathematical-Operators#average) — calculates the average of Integers emitted by an Observable and emits this average
* [**`averageDoubles( )`**](Mathematical-Operators#average) — calculates the average of Doubles emitted by an Observable and emits this average
* [**`averageFloats( )`**](Mathematical-Operators#average) — calculates the average of Floats emitted by an Observable and emits this average
* [**`averageLongs( )`**](Mathematical-Operators#average) — calculates the average of Longs emitted by an Observable and emits this average
* [**`buffer( )`**](Transforming-Observables#buffer) — periodically gather items from an Observable into bundles and emit these bundles rather than emitting the items one at a time
* [**`cache( )`**](Observable-Utility-Operators#cache) — remember the sequence of items emitted by the Observable and emit the same sequence to future Observers
* [**`cast( )`**](Transforming-Observables#cast) — cast all items from the source Observable into a particular type before reemitting them
* [**`chunkify( )`**](Blocking-Observable-Operators#chunkify) — returns an iterable that periodically returns a list of items emitted by the source Observable since the last list
* [**`combineLatest( )`**](Combining-Observables#combinelatest) — when an item is emitted by either of two Observables, combine the latest item emitted by each Observable via a specified function and emit items based on the results of this function
* [**`concat( )`**](Combining-Observables#concat) — concatenate two or more Observables sequentially
* [**`connect( )`**](Connectable-Observable-Operators#connectableobservableconnect) — instructs a Connectable Observable to begin emitting items
* [**`contains( )`**](Conditional-and-Boolean-Operators#contains) — determine whether an Observable emits a particular item or not
* [**`count( )`**](Mathematical-Operators#count-and-longcount) — counts the number of items emitted by an Observable and emits this count
* [**`create( )`**](Creating-Observables#create) — create an Observable from scratch by means of a function
* [**`debounce( )`**](Filtering-Observables#throttlewithtimeout-or-debounce) — only emit an item from the source Observable after a particular timespan has passed without the Observable emitting any other items
* [**`defaultIfEmpty( )`**](Conditional-and-Boolean-Operators#defaultifempty) — emit items from the source Observable, or emit a default item if the source Observable completes after emitting no items
* [**`defer( )`**](Creating-Observables#defer) — do not create the Observable until an Observer subscribes; create a fresh Observable on each subscription
* [**`deferFuture( )`**](Creating-Observables#deferfuture) — convert a Future that returns an Observable into an Observable, but do not attempt to get the Observable that the Future returns until an Observer subscribes
* [**`deferCancellableFuture( )`**](Creating-Observables#fromcancellablefuture-startcancellablefuture-and-defercancellablefuture-) — convert a Future that returns an Observable into an Observable in a way that monitors the subscription status of the Observable to determine whether to halt work on the Future, but do not attempt to get the returned Observable until an Observer subscribes
* [**`delay( )`**](Observable-Utility-Operators#delay) — shift the emissions from an Observable forward in time by a specified amount
* [**`dematerialize( )`**](Observable-Utility-Operators#dematerialize) — convert a materialized Observable back into its non-materialized form
* [**`distinct( )`**](Filtering-Observables#distinct) — suppress duplicate items emitted by the source Observable
* [**`distinctUntilChanged( )`**](Filtering-Observables#distinctuntilchanged) — suppress duplicate consecutive items emitted by the source Observable
* [**`doOnCompleted( )`**](Observable-Utility-Operators#dooncompleted) — register an action to take when an Observable completes successfully
* [**`doOnEach( )`**](Observable-Utility-Operators#dooneach) — register an action to take whenever an Observable emits an item
* [**`doOnError( )`**](Observable-Utility-Operators#doonerror) — register an action to take when an Observable completes with an error
* [**`elementAt( )`**](Filtering-Observables#elementat) — emit item _n_ emitted by the source Observable
* [**`elementAtOrDefault( )`**](Filtering-Observables#elementatordefault) — emit item _n_ emitted by the source Observable, or a default item if the source Observable emits fewer than _n_ items
* [**`empty( )`**](Creating-Observables#empty-error-and-never) — create an Observable that emits nothing and then completes
* [**`error( )`**](Creating-Observables#empty-error-and-never) — create an Observable that emits nothing and then signals an error
* [**`exists( )`**](Conditional-and-Boolean-Operators#exists-and-isempty) — determine whether an Observable emits any items or not
* [**`filter( )`**](Filtering-Observables#filter-or-where) — filter items emitted by an Observable
* [**`finallyDo( )`**](Observable-Utility-Operators#finallydo) — register an action to take when an Observable completes
* [**`first( )`**](Filtering-Observables#first-and-takefirst) (`Observable`) — emit only the first item emitted by an Observable, or the first item that meets some condition
* [**`first( )`**](Blocking-Observable-Operators#first-and-firstordefault) (`BlockingObservable`) — emit only the first item emitted by an Observable, or the first item that meets some condition
* [**`firstOrDefault( )`**](Filtering-Observables#firstordefault) (`Observable`) — emit only the first item emitted by an Observable, or the first item that meets some condition, or a default value if the source Observable is empty
* [**`firstOrDefault( )`**](Blocking-Observable-Operators#first-and-firstordefault) (`BlockingObservable`) — emit only the first item emitted by an Observable, or the first item that meets some condition, or a default value if the source Observable is empty
* [**`flatMap( )`**](Transforming-Observables#mapmany-or-flatmap-and-mapmanydelayerror) — transform the items emitted by an Observable into Observables, then flatten this into a single Observable
* [**`forEach( )`**](Blocking-Observable-Operators#foreach) — invoke a function on each item emitted by the Observable; block until the Observable completes
* [**`forEachFuture( )`**](Blocking-Observable-Operators#foreachfuture) — create a futureTask that will invoke a specified function on each item emitted by an Observable 
* [**`forIterable( )`**](Creating-Observables#foriterable) — apply a function to the elements of an Iterable to create Observables which are then concatenated
* [**`from( )`**](Creating-Observables#from) — convert an Iterable or a Future into an Observable
* [**`fromCancellableFuture( )`**](Creating-Observables#fromcancellablefuture-startcancellablefuture-and-defercancellablefuture-) — convert a Future into an Observable in a way that monitors the subscription status of the Observable to determine whether to halt work on the Future, but do not attempt to get the Future's value until an Observer subscribes
* [**`fromFuture( )`**](Creating-Observables#fromfuture) — convert a Future into an Observable, but do not attempt to get the Future's value until an Observer subscribes
* [**`generate( )`**](Creating-Observables#generate-and-generateabsolutetime) — create an Observable that emits a sequence of items as generated by a function of your choosing
* [**`generateAbsoluteTime( )`**](Creating-Observables#generate-and-generateabsolutetime) — create an Observable that emits a sequence of items as generated by a function of your choosing, with each item emitted at an item-specific time
* [**`getIterator( )`**](Blocking-Observable-Operators#transformations-tofuture-toiterable-and-toiteratorgetiterator) — convert the sequence emitted by the Observable into an Iterator
* [**`groupBy( )`**](Transforming-Observables#groupby-and-groupbyuntil) — divide an Observable into a set of Observables that emit groups of items from the original Observable, organized by key
* [**`groupByUntil( )`**](Transforming-Observables#groupby-and-groupbyuntil) — divide an Observable into a set of Observables that emit groups of items from the original Observable, organized by key, and opening a new set periodically
* [**`groupJoin( )`**](Combining-Observables#join-and-groupjoin) — combine the items emitted by two Observables whenever one item from one Observable falls within a window of duration specified by an item emitted by the other Observable
* [**`ignoreElements( )`**](Filtering-Observables#ignoreelements) — discard the items emitted by the source Observable and only pass through the error or completed notification
* [**`interval( )`**](Creating-Observables#interval) — create an Observable that emits a sequence of integers spaced by a given time interval
* [**`isEmpty( )`**](Conditional-and-Boolean-Operators#exists-and-isempty) — determine whether an Observable emits any items or not
* [**`join( )`**](Combining-Observables#join-and-groupjoin) — combine the items emitted by two Observables whenever one item from one Observable falls within a window of duration specified by an item emitted by the other Observable
* [**`just( )`**](Creating-Observables#just) — convert an object into an Observable that emits that object
* [**`last( )`**](Blocking-Observable-Operators#last-and-lastordefault) (`BlockingObservable`) — block until the Observable completes, then return the last item emitted by the Observable
* [**`last( )`**](Filtering-Observable-Operators#last) (`Observable`) — emit only the last item emitted by the source Observable
* [**`lastOrDefault( )`**](Blocking-Observable-Operators#last-and-lastordefault) — block until the Observable completes, then return the last item emitted by the Observable or a default item if there is no last item
* [**`latest( )`**](Blocking-Observable-Operators#latest) — returns an iterable that blocks until or unless the Observable emits an item that has not been returned by the iterable, then returns the latest such item
* [**`longCount( )`**](Mathematical-Operators#count-and-longcount) — counts the number of items emitted by an Observable and emits this count
* [**`map( )`**](Transforming-Observables#map) — transform the items emitted by an Observable by applying a function to each of them
* [**`mapMany( )`**](Transforming-Observables#mapmany-or-flatmap-and-mapmanydelayerror) — transform the items emitted by an Observable into Observables, then flatten this into a single Observable
* [**`mapManyDelayError( )`**](Transforming-Observables#mapmany-or-flatmap-and-mapmanydelayerror) — transform the items emitted by an Observable into Observables, then flatten this into a single Observable, waiting to report errors until all error-free observables have a chance to complete
* [**`mapWithIndex( )`**](Transforming-Observables#mapwithindex) — transform the items emitted by an Observable by applying a function to each of them that takes into account the index value of the item
* [**`materialize( )`**](Observable-Utility-Operators#materialize) — convert an Observable into a list of Notifications
* [**`max( )`**](Mathematical-Operators#max) — emits the maximum value emitted by a source Observable
* [**`maxBy( )`**](Mathematical-Operators#maxby) — emits the item emitted by the source Observable that has the maximum key value
* [**`merge( )`**](Combining-Observables#merge) — combine multiple Observables into one
* [**`mergeDelayError( )`**](Combining-Observables#mergedelayerror) — combine multiple Observables into one, allowing error-free Observables to continue before propagating errors
* [**`min( )`**](Mathematical-Operators#min) — emits the minimum value emitted by a source Observable
* [**`minBy( )`**](Mathematical-Operators#minby) — emits the item emitted by the source Observable that has the minimum key value
* [**`mostRecent( )`**](Blocking-Observable-Operators#mostrecent) — returns an iterable that always returns the item most recently emitted by the Observable
* [**`multicast( )`**](Connectable-Observable-Operators#observablepublish-and-observablemulticast) — represents an Observable as a Connectable Observable
* [**`never( )`**](Creating-Observables#empty-error-and-never) — create an Observable that emits nothing at all
* [**`next( )`**](Blocking-Observable-Operators#next) — returns an iterable that blocks until the Observable emits another item, then returns that item
* [**`observeOn( )`**](Observable-Utility-Operators#observeon) — specify on which Scheduler an Observer should observe the Observable
* [**`ofType( )`**](Filtering-Observables#oftype) — emit only those items from the source Observable that are of a particular class
* [**`onErrorResumeNext( )`**](Error-Handling-Operators#onerrorresumenext) — instructs an Observable to continue emitting items after it encounters an error
* [**`onErrorReturn( )`**](Error-Handling-Operators#onerrorreturn) — instructs an Observable to emit a particular item when it encounters an error
* [**`onExceptionResumeNextViaObservable( )`**](Error-Handling-Operators#onexceptionresumenextviaobservable) — instructs an Observable to continue emitting items after it encounters an exception (but not another variety of throwable)
* [**`parallel( )`**](Observable-Utility-Operators#parallel) — split the work done on the emissions from an Observable into multiple Observables each operating on its own parallel thread
* [**`parallelMerge( )`**](Combining-Observables#parallelmerge) — combine multiple Observables into smaller number of Observables
* [**`publish( )`**](Connectable-Observable-Operators#observablepublish-and-observablemulticast) — represents an Observable as a Connectable Observable
* [**`publishLast( )`**](Connectable-Observable-Operators#observablepublishlast) — represent an Observable as a Connectable Observable that emits only the last item emitted by the source Observable
* [**`range( )`**](Creating-Observables#range) — create an Observable that emits a range of sequential integers
* [**`reduce( )`**](Transforming-Observables#reduce-or-aggregate) — apply a function to each emitted item, sequentially, and emit only the final accumulated value
* [**`refCount( )`**](Connectable-Observable-Operators#connectableobservablerefcount) — makes a Connectable Observable behave like an ordinary Observable
* [**`repeat( )`**](Creating-Observables#repeat) — create an Observable that emits a particular item or sequence of items repeatedly
* [**`replay( )`**](Connectable-Observable-Operators#observablereplay) — ensures that all Observers see the same sequence of emitted items, even if they subscribe after the Observable begins emitting the items
* [**`retry( )`**](Error-Handling-Operators#retry) — if a source Observable emits an error, resubscribe to it in the hopes that it will complete without error
* [**`sample( )`**](Filtering-Observables#sample-or-throttlelast) — emit the most recent items emitted by an Observable within periodic time intervals
* [**`scan( )`**](Transforming-Observables#scan) — apply a function to each item emitted by an Observable, sequentially, and emit each successive value
* [**`sequenceEqual( )`**](Conditional-and-Boolean-Operators#sequenceequal) — test the equality of sequences emitted by two Observables
* [**`single( )`**](Blocking-Observable-Operators#single-and-singleordefault) — if the Observable completes after emitting a single item, return that item, otherwise throw an exception
* [**`singleOrDefault( )`**](Blocking-Observable-Operators#single-and-singleordefault) — if the Observable completes after emitting a single item, return that item, otherwise return a default item
* [**`skip( )`**](Filtering-Observables#skip) — ignore the first _n_ items emitted by an Observable
* [**`skipLast( )`**](Filtering-Observables#skiplast) — ignore the last _n_ items emitted by an Observable
* [**`skipUntil( )`**](Conditional-and-Boolean-Operators#skipuntil) — discard items emitted by a source Observable until a second Observable emits an item, then emit the remainder of the source Observable's items
* [**`skipWhile( )`**](Conditional-and-Boolean-Operators#skipwhile-and-skipwhilewithindex) — discard items emitted by an Observable until a specified condition is false, then emit the remainder
* [**`skipWhileWithIndex( )`**](Conditional-and-Boolean-Operators#skipwhile-and-skipwhilewithindex) — discard items emitted by an Observable until a specified condition is false, then emit the remainder
* [**`start( )`**](Creating-Observables#start) — create an Observable that emits the return value of a function
* [**`startCancellableFuture( )`**](Creating-Observables#fromcancellablefuture-startcancellablefuture-and-defercancellablefuture-) — convert a function that returns Future into an Observable that emits that Future's return value in a way that monitors the subscription status of the Observable to determine whether to halt work on the Future
* [**`startFuture( )`**](Creating-Observables#startfuture) — convert a function that returns Future into an Observable that emits that Future's return value
* [**`startWith( )`**](Combining-Observables#startwith) — emit a specified sequence of items before beginning to emit the items from the Observable
* [**`subscribeOn( )`**](Observable-Utility-Operators#subscribeon) — specify which Scheduler an Observable should use when its subscription is invoked
* [**`sum( )`**](Mathematical-Operators#sum) — adds the Integers emitted by an Observable and emits this sum
* [**`sumDoubles( )`**](Mathematical-Operators#sum) — adds the Doubles emitted by an Observable and emits this sum
* [**`sumFloats( )`**](Mathematical-Operators#sum) — adds the Floats emitted by an Observable and emits this sum
* [**`sumLongs( )`**](Mathematical-Operators#sum) — adds the Longs emitted by an Observable and emits this sum
* [**`switchOnNext( )`**](Combining-Observables#switchonnext) — convert an Observable that emits Observables into a single Observable that emits the items emitted by the most-recently emitted of those Observables
* [**`synchronize( )`**](Observable-Utility-Operators#synchronize) — force an Observable to make synchronous calls and to be well-behaved
* [**`take( )`**](Filtering-Observables#take) — emit only the first _n_ items emitted by an Observable
* [**`takeFirst( )`**](Filtering-Observables#first-and-takefirst) — emit only the first item emitted by an Observable, or the first item that meets some condition
* [**`takeLast( )`**](Filtering-Observables#takelast) — only emit the last _n_ items emitted by an Observable
* [**`takeLastBuffer( )`**](Filtering-Observables#takelastbuffer) — emit the last _n_ items emitted by an Observable, as a single list item
* [**`takeUntil( )`**](Conditional-and-Boolean-Operators#takeuntil) — emits the items from the source Observable until a second Observable emits an item
* [**`takeWhile( )`**](Conditional-and-Boolean-Operators#takewhile-and-takewhilewithindex) — emit items emitted by an Observable as long as a specified condition is true, then skip the remainder
* [**`takeWhileWithIndex( )`**](Conditional-and-Boolean-Operators#takewhile-and-takewhilewithindex) — emit items emitted by an Observable as long as a specified condition is true, then skip the remainder
* [**`then( )`**](Combining-Observables#and-then-and-when) — transform a series of `Pattern` objects via a `Plan` template
* [**`throttleFirst( )`**](Filtering-Observables#throttlefirst) — emit the first items emitted by an Observable within periodic time intervals
* [**`throttleLast( )`**](Filtering-Observables#sample-or-throttlelast) — emit the most recent items emitted by an Observable within periodic time intervals
* [**`throttleWithTimeout( )`**](Filtering-Observables#throttlewithtimeout-or-debounce) — only emit an item from the source Observable after a particular timespan has passed without the Observable emitting any other items
* [**`timeInterval( )`**](Observable-Utility-Operators#timeinterval) — emit the time lapsed between consecutive emissions of a source Observable
* [**`timeout( )`**](Filtering-Observables#timeout) — emit items from a source Observable, but issue an exception if no item is emitted in a specified timespan
* [**`timer( )`**](Creating-Observables#timer) — create an Observable that emits a single item after a given delay
* [**`timestamp( )`**](Observable-Utility-Operators#timestamp) — attach a timestamp to every item emitted by an Observable
* [**`toAsync( )`**](Creating-Observables#toasync) — convert a function into an Observable that executes the function and emits its return value
* [**`toBlockingObservable( )`**](Blocking-Observable-Operators) — transform an Observable into a BlockingObservable
* [**`toFuture( )`**](Blocking-Observable-Operators#transformations-tofuture-toiterable-and-toiteratorgetiterator) — convert the Observable into a Future
* [**`toIterable( )`**](Blocking-Observable-Operators#transformations-tofuture-toiterable-and-toiteratorgetiterator) — convert the sequence emitted by the Observable into an Iterable
* [**`toIterator( )`**](Blocking-Observable-Operators#transformations-tofuture-toiterable-and-toiteratorgetiterator) — convert the sequence emitted by the Observable into an Iterator
* [**`toList( )`**](Observable-Utility-Operators#tolist) — collect all items from an Observable and emit them as a single List
* [**`toMap( )`**](Observable-Utility-Operators#tomap-and-tomultimap) — convert the sequence of items emitted by an Observable into a map keyed by a specified key function
* [**`toMultiMap( )`**](Observable-Utility-Operators#tomap-and-tomultimap) — convert the sequence of items emitted by an Observable into an ArrayList that is also a map keyed by a specified key function
* [**`toSortedList( )`**](Observable-Utility-Operators#tosortedlist) — collect all items from an Observable and emit them as a single, sorted List
* [**`using( )`**](Observable-Utility-Operators#using) — create a disposable resource that has the same lifespan as an Observable
* [**`when( )`**](Combining-Observables#and-then-and-when) — convert a series of `Plan` objects into an Observable
* [**`where( )`**](Filtering-Observables#filter-or-where) — filter items emitted by an Observable
* [**`window( )`**](Transforming-Observables#window) — periodically subdivide items from an Observable into Observable windows and emit these windows rather than emitting the items one at a time 
* [**`zip( )`**](Combining-Observables#zip) — combine sets of items emitted by two or more Observables together via a specified function and emit items based on the results of this function