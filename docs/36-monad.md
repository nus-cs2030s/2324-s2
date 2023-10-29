# Unit 36: Monad

!!! abstract "Learning Objectives"

    Students should

    - understand what are functors and monads
    - understand the laws that a functor and monad must obey and be able to verify them

## Generalizing `Loggable<T>`

We have just created a class `Loggable<T>` with a `flatMap` method that allows us to operate on the value encapsulated inside, along with some "side information".  `Loggable<T>` follows a pattern that we have seen many times before.  We have seen this in `Maybe<T>` and `Lazy<T>`, and `InfiniteList<T>`.  Each of these classes has:

- an `of` method to initialize the value and side information.
- have a `map` method to update the value.
- have a `flatMap` method to update the value and side information[^1].

[^1]: Unfortunately our `InfiniteList<T>` has no `flatMap` method.

Different classes above have different side information that is initialized, stored, and updated when we use the `of` and `flatMap` operations.  The class may also have other methods besides the two above.  Additionally, the methods may have different name.

| Container | Side-Information |
| --------- | ---------------- |
| `Maybe<T>` | The value might be there (i.e., `Some<T>`) or might not be there (i.e., `None`) |
| `Lazy<T>` | The value has been evaluated or not |
| `Loggable<T>` | The log describing the operations done on the value |

These classes that we wrote follow certain patterns that make them well behaved when we create them with `of` and chain them with `flatMap`.  Such classes that are "_well behaved_" are examples of a programming construct called _monads_.  A monad must follow three laws, to behave well.  We will examine them later.

For now, note that we can further generalize `Loggable<T>` by abstracting the `log` into a generic type.  Previously, the `log` is always a `String`.  In this generalization, the `log` will be named as `context` with the type becoming a generic type `S`.  We can then call this abstraction a `Monad<T,S>`.

```java
class Monad<T, S> {
  private final T content; // the content (value to be operated on)
  private final S context; // the context (side-information)

    :
  
  public <U> Monad<U, S> map(Transformer<? super T, ? extends U> transformer) {
    U content = transformer.transform(this.content);
    return new Monad(content, this.context); // preserve context
  }

  public <U> Monad<U, S> flatMap(Transformer<? super T, ? extends Monad<? extends U, ? extends S>> transformer) {
    Monad<? extends U, ? extends S>> next = transformer.transform(this.content);
    return Monad.compose(this, next);
  }

  private static <T, U, S> Monad<U, S> compose(Monad<T, S> prev, Monad<? extends U, ? extends S>> next) {
      : // code omitted
  }
}
```

Note a few things, the `map` method should preserve the context while the `flatMap` method should compose the context (_by invoking_ `compose` _static method_).  The composition of the context should follow the three laws of Monad and the behavior of `map` should satisfy the two laws of Functor.

!!! warning "`flatMap` using `map`"
    A more appropriate way to define `flatMap` is to invoke `map`.  The origin of these terms is in Category Theory where the definition of a Monad is a Functor with _additional properties_.  So in theory, for something to be a Monad, it has to first be a Functor and must have satisfied the properties for `map`.

    Unfortunately, no programming language can check these properties.  So, we may actually have a structure that satisfy the three laws of Monad but does not satisfy all the laws of Functor.

    If our `flatMap` invoke `map`, then we have to make changes in our definition of `flatMap`.

    ```java
    public <U> Monad<U, S> flatMap(Transformer<? super T, ? extends Monad<? extends U, ? extends S>> fn) {
      Monad<Monad<? extends U, ? extends S>, S> monad = this.map(fn);
      return Monad.combine(monad);
    }

    private static <T, U, S> Monad<U, S> combine(Monad<Monad<? extends U, ? extends S>, S> monad) {
      U content = monad.content.content;
      S prev = monad.context;
      S next = monad.content.context;

      return new Monad<>(content, Monad.compose(prev, next));
    }
    ```

    Now we are invoking `combine` which takes in a nested monad!  But this is a special monad because the two contexts are there.  Recap that `map` preserves the context so `monad.context` is the old context.  Additionally, the content is now another Monad which has additional content and context.

    If things are confusing, do not worry, you are not expected to understand the entire details (_especially not with that horrible type_).  You just need to understand the three laws (_and how to check for them_) as well as the two laws (_and how to check for them_).

## Three Laws of Monad

The three laws below are all related to the context.  In particular, we need to ensure that the `compose` method (_i.e., the composition of_ `context`).  In particular, we want the `of` method to add "identity" context and we want `flatMap` to be associative with respect to the context.

So, the `of` method in a monad should create a new monad by initializing our monad with a value and its side information should be whatever is considered an "empty" side-information (_more formally, this is the identity element_).   For instance, in our `Loggable<T>`,

```Java
public static <T> Loggable<T> of(T value) {
  return new Loggable<>(value, "");
}
```

Now, we can look at this in more details.

### Identity Laws

Let's consider the lambda that we wish to pass into `flatMap`  -- such a lambda takes in a value, compute it, and wrap it in a "new" monad, together with the correponding side information.  For instance,

```Java
Loggable<Integer> incrWithLog(int x) {
  return new Loggable<>(incr(x), "incr " + x + "; ");
}
```

What should we expect when we take a fresh new monad `Loggable.of(4)` and call `flatMap` with a function `incrWithLog`?  Since `Loggable.of(4)` is new with no operation performed on it yet, calling 
```Java
Loggable.of(4).flatMap(x -> incrWithLog(x)) 
```

should just result in the same value exactly as calling `incrWithLog(4)`.  So, we expect that, after calling the above, we have a `Loggable` with a value 5 and a log message of `"incr 4"`.

Our `of` method should not do anything extra to the value and side information -- it should simply wrap the value 4 into the `Loggable`.  Our `flatMap` method should not do anything extra to the value and the side information, it should simply apply the given lambda expression to the value.

Now, suppose we take an instance of `Loggable`, called `logger`, that has already been operated on one or more times with `flatMap`, and contain some side information.  What should we expect when we call:
```Java
logger.flatMap(x -> Loggable.of(x))
```

Since `of` should behave like an identity, it should not change the value or add extra side information.  The `flatMap` above should do nothing and the expression above should be the same as `logger`.

What we have just described above is called the _left identity law_ and the _right identity law_ of monads.  To be more general, let `Monad` be a type that is a monad and `monad` be an instance of it.

The left identity law says:

- `Monad.of(x).flatMap(x -> f(x))` must be the same as `f(x)`

The right identity law says:

- `monad.flatMap(x -> Monad.of(x))` must be the same as `monad`

So the identity laws is not just a single law, it is actually two laws.  A good thing about this is that if both left identity and right identity element exists, it must be the same value.

### Associative Law

Let's now go back to the original `incr` and `abs` functions for a moment.  To compose the functions, we can write `abs(incr(x))`, explicitly one function after another.  Or we can compose them as another function: 
```Java
int absIncr(int x) {
  return abs(incr(x));
}
```

and call it `absIncr(x)`.  The effects should be exactly the same.  It does not matter if we group the functions together into another function before applying it to a value x.

Recall that after we build our `Loggable` class, we were able to compose the functions `incr` and `abs` by chaining the `flatMap`:

```Java
Loggable.of(4)
        .flatMap(x -> incrWithLog(x))
        .flatMap(x -> absWithLog(x))
```

We should get the resulting value as `abs(incr(4))`, along with the appropriate log messages.

Another way to call `incr` and then `abs` is to write something like this:
```Java
Loggable<Integer> absIncrWithLog(int x) {
  return incrWithLog(x).flatMap(y -> absWithLog(y));
}
```

We have composed the methods `incrWithLog` and `absWithLog` and grouped them under another method.  Now, if we call:
```Java
Loggable.of(4)
    .flatMap(x -> absIncrWithLog(x))
```

The two expressions must have exactly the same effect on the value and its log message.

This example leads us to the third law of monads: regardless of how we group that calls to `flatMap`, their behaviour must be the same.  This law is called the _associative law_.  More formally, it says:

- `monad.flatMap(x -> f(x)).flatMap(x -> g(x))` must be the same as `monad.flatMap(x -> f(x).flatMap(y -> g(y)))`

## A Counter Example

If our monads follow the laws above, we can safely write methods that receive a monad from others, operate on it, and return it to others.  We can also safely create a monad and pass it to the clients to operate on.  Our clients can then call our methods in any order and operate on the monads that we create, and the effect on its value and side information is as expected.

Let's try to make our `Loggable` misbehave a little.  Suppose we change our `Loggable<T>` to be as follows:

```Java hl_lines="12 17"
// version 0.3 (NOT a monad)
class Loggable<T> {
  private final T value;
  private final String log;

  private Loggable(T value, String log) {
    this.value = value;
    this.log = log;
  }

  public static <T> Loggable<T> of(T value) {
    return new Loggable<>(value, "Logging starts: ");
  }

  public <R> Loggable<R> flatMap(Transformer<? super T, ? extends Loggable<? extends R>> transformer) {
    Loggable<? extends R> logger = transformer.transform(this.value);
    return new Loggable(logger.value, logger.log + this.log + "\n");
  }

  public String toString() {
    return "value: " + this.value + ", log: " + this.log;
  }
}
```

Our `of` adds a little initialization message.  Our `flatMap` adds a little new line before appending with the given log message.  Now, our `Loggable<T>` is not that well behaved anymore.

Suppose we have two methods `foo` and `bar`, both take in an `x` and perform a series of operations on `x`.  Both returns us a `Loggable` instance on the final value and its log.

```Java
Loggable<Integer> foo(int x) {
  return Loggable.of(x)
                 .flatMap(...)
                 .flatMap(...)
                   :
  ;
}
Loggable<Integer> bar(int x) {
  return Loggable.of(x)
                 .flatMap(...)
                 .flatMap(...)
                   :
  ;
}
```

Now, we want to perform the sequence of operations done in `foo`, followed by the sequence of operations done in `bar`.  So we called:
```Java
foo(4).flatMap(x -> bar(x))
```

We will find that the string `"Logging starts"` appears twice in our logs and there is now an extra blank line in the log file!

## Two Laws of Functors

We will end this unit with a brief discussion on _functors_, another common abstraction in functional-style programming.  A functor is a simpler construction than a monad in that it only ensures lambdas can be applied sequentially to the value, without worrying about side information.

Recall that when we build our `Loggable<T>` abstraction, we add a `map` that only updates the value but changes nothing to the side information.  One can think of a functor as an abstraction that supports `map`.

A functor needs to adhere to two laws:

- preserving identity: `functor.map(x -> x)` is the same as `functor`
- preserving composition: `functor.map(x -> f(x)).map(x -> g(x))` is the same as `functor.map(x -> g(f(x))`. 

Note that we can also infer what they should do to the context (_if any_).  If the `map` method actually modifies the context, then applying the identity function `x -> x` would have modified the context.  So it would no longer be the same as the original `functor`.  So we know that `map` cannot modify context.  This also means that `map` should not even change the context into the identity context created using the `of` method.

Our classes from `cs2030s.fp`, `Lazy<T>`, `Maybe<T>`, and `InfiniteList<T>` are functors as well.

## Monads and Functors in Other Languages

Such abstractions are common in other languages.  In Scala, for instance, the collections (list, set, map, etc.) are monads.  In pure functional languages like Haskell, monads are one of the fundamental building blocks.
