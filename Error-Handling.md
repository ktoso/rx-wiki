_**Work in progress...**_

An Observable typically does not _throw_ exceptions. Instead it notifies any observers that an unrecoverable error has occurred by terminating the Observable sequence with an `onError` notification.

There are some exceptions to this. For example, if the `onError()` call _itself_ fails, the Observable will not attempt to notify the observer of this by again calling `onError` but will throw a `RuntimeException` or an `OnErrorNotImplementedException`.

# Techniques for recovering from onError notifications

So rather than _catch_ exceptions, your observer or operator should more typically respond to `onError` notifications of exceptions. There are also a variety of Observable operators that you can use to react to or recover from `onError` notifications from Observables. For example, you might use an operator to:

1. swallow the error and switch over to a backup Observable to continue the sequence
1. swallow the error and emit a default item
1. swallow the error and immediately try to restart the failed Observable
1. swallow the error and try to restart the failed Observable after some back-off interval

You can use the operators described in [[Error Handling Operators]] to implement these strategies.

# RxJava-specific exceptions and what do do about them

<table>
 <thead>
  <tr><th>Exception</th><th>what it indicates</th></tr>
 </thead>
 <tbody>
  <tr><td><code>CompositeException</code></td><td>This indicates that more than one exception occurred. You can use the exception&#8217;s <code>getMessage()</code> method to retrieve a concatenation of the exceptions that make up the composite.</td></tr>
  <tr><td><code>MissingBackpressureException</code></td><td>This indicates that a Subscriber or operator attempted to apply reactive pull backpressure to an Observable that does not implement it. See [[Backpressure]] for work-arounds for Observables that do not implement reactive pull backpressure.</td></tr>
  <tr><td><code>OnErrorFailedException</code></td><td>This indicates that an Observable tried to call its observer&#8217;s <code>onError()</code> method, but that method itself threw an exception.</td></tr>
  <tr><td><code>OnErrorNotImplementedException</code></td><td>This indicates that an Observable tried to call its observer&#8217;s <code>onError()</code> method, but that no such method existed. You can eliminate this by either fixing the Observable so that it no longer reaches an error condition, by implementing an `onError` handler in the observer, or by intercepting the `onError` notification before it reaches the observer by using one of the above-described operators.</td></tr>
  <tr><td><code>OnErrorThrowable</code></td><td>Observers pass throwables of this sort into their observers&#8217; <code>onError()</code> handlers. A Throwable of this variety contain more information about the error and about the Observable-specific state of the system at the time of the error than does a standard <code>Throwable</code>.</td></tr>
 </tbody>
</table>

_**Work in progress...**_