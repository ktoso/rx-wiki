# Introduction

Writing operators, source-like (`fromAsync`) or intermediate-like (`flatMap`) **has always been a hard task to do in RxJava**. There are many rules to obey, many cases to consider but at the same time, many (legal) shortcuts to take to build a well performing code. Now writing an operator specifically for 2.x is 10 times harder than for 1.x. If you want to exploit all the advanced, 4th generation features, that's even 2-3 times harder on top (so 30 times harder in total).

*(If you have been following [my blog](http://akarnokd.blogspot.hu/) about RxJava internals, writing operators is maybe only 2 times harder than 1.x; some things have moved around, some tools popped up while others have been dropped but there is a relatively straight mapping from 1.x concepts and approaches to 2.x concepts and approaches.)*

In this article, I'll describe the how-to's from the perspective of a developer who skipped the 1.x knowledge base and basically wants to write operators that conforms the Reactive-Streams specification as well as RxJava 2.x's own extensions and additional expectations/requirements.

# Atomics, serialization, deferred actions

TBD

# Backpressure and cancellation

TBD

# Operator fusion

TBD

# Example implementations

TBD

## `map` + `filter` hybrid

TBD

## Ordered `merge`

TBD
