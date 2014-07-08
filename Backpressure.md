_**Work in progress...**_

In RxJava it is not difficult to get into a situation in which an Observable is emitting items more rapidly than an operator or subscriber can consume them. This presents the problem of what to do with such a growing backlog of unconsumed items.

For example, imagine using the `zip` operator to zip together two infinite Observables, one of which emits items twice as frequently as the other. The `zip` operator, to perform as advertised, will have to maintain an ever-expanding buffer of items emitted by the faster Observable to combine with items emitted by the slower one. This could cause RxJava to seize an unwieldy amount of system resources.

You can tell RxJava how you want it to handle cases like these. RxJava is capable of exerting _backpressure_ on Observables. This page tells you how the backpressure options work, and also how you can design your own Observables and Observable operators to respect backpressure requests.

_**Work in progress...**_