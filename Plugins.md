Plugins allow you to modify the default behavior of RxJava in several respects.

# RxJavaDefaultSchedulers

This plugin allows you to override the default computation, i/o, and new thread Schedulers with Schedulers of your choosing.  To do this, extend the class `RxJavaDefaultSchedulers` and override these methods:

* `Scheduler getComputationScheduler()`
* `Scheduler getIOScheduler()`
* `Scheduler getNewThreadScheduler()`

Then follow these steps:

# Create an object of the new `RxJavaDefaultSchedulers` subclass you have implemented.
# Obtain the global `RxJavaPlugins` instance via `RxJavaPlugins.getInstance()`.
# Pass your default schedulers object to the `RxJavaPlugins.registerDefaultSchedulers()` method.

When you do this, RxJava will begin to use the Schedulers returned by your methods rather than its built-in defaults.

# RxJavaErrorHandler
# RxJavaObservableExecutionHook
