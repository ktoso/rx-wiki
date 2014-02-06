_Content TBD_

## Default Schedulers for RxJava Observable operators

Some Observable operators in RxJava have alternate forms that allow you to set which Scheduler the operator will use for (at least some part of) its operation. For these operators, if you do not set the Scheduler, the operator will use the default `computation` Scheduler.

Other operators do not have a form that permits you to set their Schedulers. Some of these, like `startWith`, `empty`, `error`, `from`, `just`, `merge`, and `range` do not use a Scheduler. A few others use particular schedulers, as in the following table:
<table>
 <thead>
  <tr><th>operator</th><th>Scheduler</th></tr>
 </thead>
 <tbody>
  <tr><td><code>repeat</code></td><td><code>currentThread</code></td></tr>
  <tr><td><code>timeInterval</code></td><td><code>immediate</code></td></tr>
  <tr><td><code>timestamp</code></td><td><code>immediate</code></td></tr>
  <tr><td><code>parallelMerge</code></td><td><code>currentThread</code></td></tr>
 </tbody>
</table>