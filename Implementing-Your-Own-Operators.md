You can implement your own Observable operators. This page shows you how.

# Chaining Your Custom Operators with Standard RxJava Operators

The following example shows how you can chain a custom operator (in this example: `myOperator`) along with standard RxJava operators by using the `lift( )` operator:
```groovy
Observable foo = barObservable.ofType(Integer).map({it*2}).lift(new myOperator<T>()).map({"transformed by myOperator: " + it});
```
The following section will show how to form the scaffolding of your operator so that it will work correctly with `lift( )`.

# Implementing Your Operator

Define your operator as a public class, like so:
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

# Other Considerations

* Your operator should take care to check its Subscriber's `isUnsubscribed( )` status before it emits any items to (or sends any notifications to) the Subscriber. Do not waste the time to generate an item that no Subscriber is interested in seeing.
* Your operator should obey the core tenets of the Observable contract:
** It may call a Subscriber's `onNext( )` method any number of times, but these calls must be non-overlapping.
** It may call either a Subscriber's `onCompleted( )` or `onError( )` method, but not both, exactly once, and it may not subsequently call a Subscriber's `onNext( )` method.