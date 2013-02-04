RxJava is still a work in progress and has a long list of work documented in the [[Issues|https://github.com/Netflix/RxJava/issues]].

Netflix is working on the 0.6 tasks. We would appreciate help on anything scheduled for version 0.7 or later or of course bugs.

If you wish to contribute we would ask that you:

- review existing code and comply with existing patterns and idioms
- include unit tests
- stick to Rx contracts as defined by the Rx.Net implementation when porting operators (each issue attempts to reference the correct documentation from MSDN)

Note that anything involving schedulers will need to wait until [[Issue 19|https://github.com/Netflix/RxJava/issues/19]] is completed to establish the correct approach that fits the Rx model while being appropriate for Java where ExecutorService is likely to be the right abstraction.

Information about licensing can be found at: [[CONTRIBUTING|https://github.com/Netflix/RxJava/blob/master/CONTRIBUTING.md]].
