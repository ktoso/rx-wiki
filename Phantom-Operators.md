This page includes write-ups of Observable operators that were proposed at some point but then dropped before the 1.0 release.  They're here in case they get resurrected, in which case we can slide them back into the main documentation set.

* [**`forEachFuture( )`**](Phantom-Operators#wiki-foreachfuture) — create a futureTask that will invoke a specified function on each item emitted by an Observable
* [**`chunkify( )`**](Phantom-Operators#wiki-chunkify) — returns an iterable that periodically returns a list of items emitted by the source Observable since the last list

***

## forEachFuture( )
#### create a futureTask that will invoke a specified function on each item emitted by an Observable 
[[images/rx-operators/B.forEachFuture.png]]

The `forEachFuture( )` returns a `FutureTask` for each item emitted by the source Observable (or each item and each notification) that, when executed, will apply a function you specify to each such item (or item and notification).

***

## chunkify( )
#### returns an iterable that periodically returns a list of items emitted by the source Observable since the last list
[[images/rx-operators/B.chunkify.png]]

The `chunkify( )` operator represents a blocking observable as an Iterable, that, each time you iterate over it, returns a list of items emitted by the source Observable since the previous iteration. These lists may be empty if there have been no such items emitted.

***
