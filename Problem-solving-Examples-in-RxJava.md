This page will present some elementary RxJava puzzles and walk through some solutions (using the Groovy language implementation of RxJava) as a way of introducing you to some of the RxJava operators.

# Project Euler problem #1

There used to be a site called "Project Euler" that presented a series of mathematical computing conundrums (some fairly easy, others quite baffling) and challenged people to solve them. The first one was a sort of warm-up exercise:

> If we list all the natural numbers below 10 that are multiples of 3 or 5, we get 3, 5, 6 and 9. The sum of these multiples is 23. Find the sum of all the multiples of 3 or 5 below 1000.

There are several ways we could go about this with RxJava.  We might, for instance, begin by going through all of the natural numbers below 1000 with [`range`](Creating-Observables#range) and then [`filter`](Filtering-Observables#filter) out those that are not a multiple either of 3 or of 5:
````groovy
def threesAndFives = Observable.range(1,999).filter({ !((it % 3) && (it % 5)) });
````
Or, we could generate two Observable sequences, one containing the multiples of three and the other containing the multiples of five (by [`map`](https://github.com/Netflix/RxJava/wiki/Transforming-Observables#map)ping each value onto its appropriate multiple), making sure to only generating new multiples while they are less than 1000 (the [`takeWhile`](Conditional-and-Boolean-Operators#takewhile-and-takewhilewithindex) operator will help here), and then [`merge`](Combining-Observables#merge) these sets:
````groovy
def threes = Observable.range(1,999).map({it*3}).takeWhile({it<1000});
def fives = Observable.range(1,999).map({it*5}).takeWhile({it<1000});
def threesAndFives = Observable.merge(threes, fives).distinct();
````
Don't forget the [`distinct`](Filtering-Observables#distinct) operator here, otherwise merge will duplicate numbers like 15 that are multiples of both 5 and 3.

Next, we want to sum up the numbers in the resulting sequence. If you have installed the optional `rxjava-math` module, this is elementary: just use an operator like [`sumInteger` or `sumLong`](Mathematical-and-Aggregate-Operators#suminteger-sumlong-sumfloat-and-sumdouble) on the `threesAndFives` Observable. But what if you don't have this module? How could you use standard RxJava operators to sum up a sequence and emit that sum?

There are a number of operators that reduce a sequence emitted by a source Observable to a single value emitted by the resulting Observable. Most of the ones that are not in the `rxjava-math` module emit boolean evaluations of the sequence; we want something that can emit a number. The [`reduce`](Mathematical-and-Aggregate-Operators#reduce) operator will do the job:
````groovy
def summer = threesAndFives.reduce(0, { a, b -> a+b });
````
Here is how `reduce` gets the job done. It starts with 0 as a seed. Then, with each item that `threesAndFives` emits, it calls the closure `{ a, b -> a+b }`, passing it the current seed value as `a` and the emission as `b`. The closure adds these together and returns that sum, and `reduce` uses this returned value to overwrite its seed. When `threesAndFives` completes, `reduce` emits the final value returned from the closure as its sole emission:
<table>
 <thead>
  <tr><th>iteration</th><th>seed</th><th>emission</th><th>reduce</th></tr>
 </thead>
 <tbody>
  <tr><td>1</td><td>0</td><td>3</td><td>3</td></tr>
  <tr><td>2</td><td>3</td><td>5</td><td>8</td></tr>
  <tr><td>3</td><td>8</td><td>6</td><td>14</td></tr>
  <tr><td colspan="4"><center>&hellip;</center></td></tr>
  <tr><td>466</td><td>232169</td><td>999</td><td>233168</td></tr>
 </tbody>
</table>
Finally, we want to see the result. This means we must [subscribe](Observable#onnext-oncompleted-and-onerror) to the Observable we have constructed:
````groovy
summer.subscribe({println(it);});
````
# Generate the Fibonacci Sequence

How could you create an Observable that emits [the Fibonacci sequence](http://en.wikipedia.org/wiki/Fibonacci_number)?