* [**`aggregate( )`**](Transforming-Observables#reduce-or-aggregate) — apply a function to each emitted item, sequentially, and emit only the final accumulated value
* [**`amb( )`**](Combining-Observables#amb) — given two or more source Observables, emits all of the items from the first of these Observables to emit an item
* [**`buffer( )`**](Transforming-Observables#buffer) — periodically gather items from an Observable into bundles and emit these bundles rather than emitting the items one at a time 
* [**`cast( )`**](Transforming-Observables#cast) — cast all items from the source Observable into a particular type before reemitting them
* [**`combineLatest( )`**](Combining-Observables#combinelatest) — when an item is emitted by either of two Observables, combine the latest item emitted by each Observable via a specified function and emit items based on the results of this function
* [**`concat( )`**](Combining-Observables#concat) — concatenate two or more Observables sequentially
* [**`create( )`**](Creating-Observables#create) — create an Observable from scratch by means of a function
* [**`debounce( )`**](Filtering-Observables#throttlewithtimeout-or-debounce) — only emit an item from the source Observable after a particular timespan has passed without the Observable emitting any other items
* [**`defaultIfEmpty( )`**](Transforming-Observables#defaultifempt) — emit items from the source Observable, or emit a default item if the source Observable completes after emitting no items
* [**`defer( )`**](Creating-Observables#defer) — do not create the Observable until an Observer subscribes; create a fresh Observable on each subscription
* [**`distinct( )`**](Filtering-Observables#distinct) — suppress duplicate items emitted by the source Observable
* [**`distinctUntilChanged( )`**](Filtering-Observables#distinctuntilchanged) — suppress duplicate consecutive items emitted by the source Observable
* [**`elementAt( )`**](Filtering-Observables#elementat) — emit item _n_ emitted by the source Observable
* [**`elementAtOrDefault( )`**](Filtering-Observables#elementatordefault) — emit item _n_ emitted by the source Observable, or a default item if the source Observable emits fewer than _n_ items
* [**`empty( )`**](Creating-Observables#empty-error-and-never) — create an Observable that emits nothing and then completes
* [**`error( )`**](Creating-Observables#empty-error-and-never) — create an Observable that emits nothing and then signals an error
* [**`filter( )`**](Filtering-Observables#filter-or-where) — filter items emitted by an Observable
* [**`first( )`**](Filtering-Observables#first) — emit only the first item emitted by an Observable, or the first item that meets some condition
* [**`firstOrDefault( )`**](Filtering-Observables#firstordefault) — emit only the first item emitted by an Observable, or the first item that meets some condition, or a default value if the source Observable is empty
* [**`flatMap( )`**](Transforming-Observables#mapmany-or-flatmap-and-mapmanydelayerror) — transform the items emitted by an Observable into Observables, then flatten this into a single Observable
* [**`from( )`**](Creating-Observables#from) — convert an Iterable or a Future into an Observable
* [**`groupBy( )`**](Transforming-Observables#groupby) — divide an Observable into a set of Observables that emit groups of items from the original Observable, organized by key
* [**`ignoreElements( )`**](Filtering-Observables#ignoreelements) — discard the items emitted by the source Observable and only pass through the error or completed notification
* [**`interval( )`**](Creating-Observables#interval) — create an Observable that emits a sequence of integers spaced by a given time interval
* [**`just( )`**](Creating-Observables#just) — convert an object into an Observable that emits that object
* [**`map( )`**](Transforming-Observables#map) — transform the items emitted by an Observable by applying a function to each of them
* [**`mapMany( )`**](Transforming-Observables#mapmany-or-flatmap-and-mapmanydelayerror) — transform the items emitted by an Observable into Observables, then flatten this into a single Observable
* [**`mapManyDelayError( )`**](Transforming-Observables#mapmany-or-flatmap-and-mapmanydelayerror) — transform the items emitted by an Observable into Observables, then flatten this into a single Observable, waiting to report errors until all error-free observables have a chance to complete
* [**`mapWithIndex( )`**](Transforming-Observables#mapwithindex) — transform the items emitted by an Observable by applying a function to each of them that takes into account the index value of the item
* [**`merge( )`**](Combining-Observables#merge) — combine multiple Observables into one
* [**`mergeDelayError( )`**](Combining-Observables#mergedelayerror) — combine multiple Observables into one, allowing error-free Observables to continue before propagating errors
* [**`never( )`**](Creating-Observables#empty-error-and-never) — create an Observable that emits nothing at all
* [**`ofClass( )`**](Filtering-Observables#ofclass) — emit only those items from the source Observable that are of a particular class
* [**`range( )`**](Creating-Observables#range) — create an Observable that emits a range of sequential integers
* [**`reduce( )`**](Transforming-Observables#reduce-or-aggregate) — apply a function to each emitted item, sequentially, and emit only the final accumulated value
* [**`sample( )`**](Filtering-Observables#sample-or-throttlelast) — emit the most recent items emitted by an Observable within periodic time intervals
* [**`scan( )`**](Transforming-Observables#scan) — apply a function to each item emitted by an Observable, sequentially, and emit each successive value
* [**`skip( )`**](Filtering-Observables#skip) — ignore the first _n_ items emitted by an Observable
* [**`skipLast( )`**](Filtering-Observables#skiplast) — ignore the last _n_ items emitted by an Observable
* [**`skipWhile( )` and `skipWhileWithIndex( )`**](Filtering-Observables#skipwhile-and-skipwhilewithindex) — discard items emitted by an Observable until a specified condition is false, then emit the remainder
* [**`startWith( )`**](Combining-Observables#startwith) — emit a specified sequence of items before beginning to emit the items from the Observable
* [**`switchOnNext( )`**](Combining-Observables#switchonnext) — convert an Observable that emits Observables into a single Observable that emits the items emitted by the most-recently emitted of those Observables
* [**`take( )`**](Filtering-Observables#take) — emit only the first _n_ items emitted by an Observable
* [**`takeLast( )`**](Filtering-Observables#takelast) — only emit the last _n_ items emitted by an Observable
* [**`takeUntil( )`**](Combining-Observables#takeuntil) — emits the items from the source Observable until a second Observable emits an item
* [**`takeWhile( )` and `takeWhileWithIndex( )`**](Filtering-Observables#takewhile-and-takewhilewithindex) — emit items emitted by an Observable as long as a specified condition is true, then skip the remainder
* [**`throttleFirst( )`**](Filtering-Observables#throttlefirst) — emit the first items emitted by an Observable within periodic time intervals
* [**`throttleLast( )`**](Filtering-Observables#sample-or-throttlelast) — emit the most recent items emitted by an Observable within periodic time intervals
* [**`throttleWithTimeout( )`**](Filtering-Observables#throttlewithtimeout-or-debounce) — only emit an item from the source Observable after a particular timespan has passed without the Observable emitting any other items
* [**`timeout( )`**](Filtering-Observables#timeout) — emit items from a source Observable, but issue an exception if no item is emitted in a specified timespan
* [**`where( )`**](Filtering-Observables#filter-or-where) — filter items emitted by an Observable
* [**`window( )`**](Transforming-Observables#window) — periodically subdivide items from an Observable into Observable windows and emit these windows rather than emitting the items one at a time 
* [**`zip( )`**](Combining-Observables#zip) — combine sets of items emitted by two or more Observables together via a specified function and emit items based on the results of this function
