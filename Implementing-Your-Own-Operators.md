You can implement your own Observable operators. This page shows you how.

If your operator is designed to *originate* an Observable, rather than to transform or react to a source Observable, use the [`create( )`](Creating-Observables#wiki-create) method rather than trying to implement `Observable` manually.  Otherwise, follow the instructions below.

If your operator is designed to transform the items emitted by a source Observable, follow the instructions under _Transformational Operators_ below. If your operator is designed to alter the source Observable as a whole, follow the instruction under _Compositional Operators_ below.

(**Note:** in Xtend, a Groovy-like language, you can implement your operators as _extension methods_ and can thereby chain them directly without using the methods described on this page. See [RxJava and Xtend](http://mnmlst-dvlpr.blogspot.de/2014/07/rxjava-and-xtend.html) for details.)

# Compositional Operators

The following example shows how you can use the `lift( )` operator to chain your custom operator (in this example: `myOperator`) alongside standard RxJava operators like `ofType` and `map`:
```groovy
fooObservable = barObservable.ofType(Integer).map({it*2}).lift(new myOperator<T>()).map({"transformed by myOperator: " + it});
```
The following section shows how you form the scaffolding of your operator so that it will work correctly with `lift( )`.

# Implementing Your Operator

Define your operator as a public class that implements the [`Operator`](http://netflix.github.io/RxJava/javadoc/rx/Observable.Operator.html) interface, like so:
```java
public class myOperator<T> implements Operator<T> {
  public myOperator( /* any necessary params here */ ) {
    /* any necessary initialization here */
  }

  @Override
  public Subscriber<? super T> call(final Subscriber<? super T> s) {
    return new Subscriber<t>(s) {
      @Override
      public void onCompleted() {
        /* add your own onCompleted behavior here, or just pass the completed notification through: */
        if(!s.isUnsubscribed()) {
          s.onCompleted();
        }
      }

      @Override
      public void onError(Throwable t) {
        /* add your own onError behavior here, or just pass the error notification through: */
        if(!s.isUnsubscribed()) {
          s.onError(t);
        }
      }

      @Override
      public void onNext(T item) {
        /* this example performs some sort of simple transformation on each incoming item and then passes it along */
        if(!s.isUnsubscribed()) {
          transformedItem = myOperatorTransformOperation(item);
          s.onNext(transformedItem);
        }
      }
    };
  }
}
``` 

### Other Considerations

* Your operator should check [its Subscriber's `isUnsubscribed( )` status](Observable#unsubscribing) before it emits any item to (or sends any notification to) the Subscriber. Do not waste time generating items that no Subscriber is interested in seeing.
* Your operator should obey the core tenets of the Observable contract:
  * It may call a Subscriber's [`onNext( )`](Observable#onnext-oncompleted-and-onerror) method any number of times, but these calls must be non-overlapping.
  * It may call either a Subscriber's [`onCompleted( )`](Observable#onnext-oncompleted-and-onerror) or [`onError( )`](Observable#onnext-oncompleted-and-onerror) method, but not both, exactly once, and it may not subsequently call a Subscriber's [`onNext( )`](Observable#onnext-oncompleted-and-onerror) method.
  * If you are unable to guarantee that your operator conforms to the above two tenets, you can add the [`serialize( )`](Observable-Utility-Operators#serialize) operator to it to force the correct behavior.
* Do not block within your operator.
* It is usually best that you compose new operators by combining existing ones, to the extent that this is possible, rather than reinventing the wheel. RxJava itself does this with some of its standard operators, for example:
  * [`first( )`](Filtering-Observables#wiki-first-and-takefirst) is defined as [`take(1)`](Filtering-Observables#wiki-take)`.`[`single( )`](Observable-Utility-Operators#wiki-single-and-singleordefault)
  * [`ignoreElements( )`](Filtering-Observables#wiki-ignoreelements) is defined as [`filter(alwaysFalse( ))`](Filtering-Observables#wiki-filter)
  * [`reduce(a)`](Mathematical-and-Aggregate-Operators#wiki-reduce) is defined as [`scan(a)`](Transforming-Observables#wiki-scan)`.`[`last( )`](Filtering-Observables#wiki-last)
* If your operator uses functions or lambdas that are passed in as parameters (predicates, for instance), note that these may be sources of exceptions, and be prepared to catch these and notify subscribers via `onError( )` calls.
  * Some exceptions are considered "fatal" and for them there's no point in trying to call `onError( )` because that will either be futile or will just compound the problem. You can use the `Exceptions.throwIfFatal(throwable)` method to filter out such fatal exceptions and rethrow them rather than try to notify about them.
* In general, notify subscribers of error conditions immediately, rather than making an effort to emit more items first.

# Transformational Operators

TBD