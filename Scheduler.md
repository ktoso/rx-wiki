## Introduction

_Content TBD_

## Varieties of Scheduler

You obtain a Scheduler from the factory methods described in the `Schedulers` class. The following table shows the varieties of Scheduler that are available to you by means of these methods:

<table>
 <thead>
  <tr><th><code>Scheduler</cote></th><th>purpose</th></tr>
 </thead>
 <tbody>
  <tr><td><code>Schedulers.computation(&#8239;)</code></td><td>meant for computational work such as event-loops and callback processing; do not use this scheduler for I/O (use <code>Schedulers.io(&#8239;)</code> instead)</td></tr>
  <tr><td><code>Schedulers.currentThread(&#8239;)</code></td><td>queues work to begin on the current thread after any already-queued work</td></tr>
  <tr><td><code>Schedulers.executor(&#8239;)</code></td><td>queues work to be done on either an <code>Executor</code> or <code>ScheduledExecutorService</code> (Note that if you use an <code>Executor</code> instead of a <code>ScheduledExecutorService</code> then the Scheduler will use a system-wide <code>Timer</code> to handle delayed events.)</td></tr>
  <tr><td><code>Schedulers.immediate(&#8239;)</code></td><td>schedules work to begin immediately in the current thread</td></tr>
  <tr><td><code>Schedulers.io(&#8239;)</code></td><td>meant for I/O-bound work such as asynchronous performance of blocking I/O, this scheduler is backed by an Executor thread-pool that will grow as needed; for ordinary computational work, switch to <code>Schedulers.computation(&#8239;)</code></td></tr>
  <tr><td><code>Schedulers.newThread(&#8239;)</code></td><td>creates a new thread for each unit of work</td></tr>
 </tbody>
</table>
## Default Schedulers for RxJava Observable operators

Some Observable operators in RxJava have alternate forms that allow you to set which Scheduler the operator will use for (at least some part of) its operation. For these operators, if you do not set the Scheduler, the operator will use the default `computation` Scheduler.

Other operators do not have a form that permits you to set their Schedulers. Some of these, like `startWith`, `empty`, `error`, `from`, `just`, `merge`, and `range` do not use a Scheduler. A few others use particular schedulers, as in the following table:
<table>
 <thead>
  <tr><th>operator</th><th>Scheduler</th></tr>
 </thead>
 <tbody>
  <tr><td><code>parallelMerge</code></td><td><code>currentThread</code></td></tr>
  <tr><td><code>repeat</code></td><td><code>currentThread</code></td></tr>
  <tr><td><code>timeInterval</code></td><td><code>immediate</code></td></tr>
  <tr><td><code>timestamp</code></td><td><code>immediate</code></td></tr>
 </tbody>
</table>