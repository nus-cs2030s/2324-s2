# Unit 35: Loggable

!!! abstract "Learning Objectives"

    Students should

    - understand why we need the `flatMap` method.

## The Story So Far

So far in the class, we have seen very general abstractions that support the `flatMap` operation.  But, it is not clear where this operation comes from, why is it fundamental, nor why is it useful[^1]

In this unit, we are going to build a general abstraction step-by-step, get stuck at some point, and see how `flatMap` comes to our rescue, and hopefully, through this exercise, you will get some appreciation of `flatMap`.

## Function Composition

Let's start with some methods that we wish to operate over `int`.  Let's use some trivial functions so that we don't get distracted by its details.

```Java
int incr(int x) {
  return x + 1;
}

int abs(int x) {
  return x > 0 ? x : -x;
}
```

These methods are pure functions without side effects, they take in one argument and produce a result. 

Just like mathematical functions, we can compose them together in arbitrary order to form more complex operations.

```Java
incr(abs(-4));
abs(incr(incr(5)));
```

## `Loggable` with `Pair`

Suppose now we want to return not only an `int`, but some additional information related to the operation on `int`.  For instance, let's suppose we want to return a string describing the operation (_for logging_).  Java does not support returning multiple values, so let's return a `Pair`.

```Java
Pair<Integer,String> incrWithLog(int x) {
  return Pair.of(incr(x), "; incr " + x);
}

Pair<Integer,String> absWithLog(int x) {
  return Pair.of(abs(x), "; abs " + x);
}
```

Now, we can't compose the methods as cleanly as before.  This is because the return value of `absWithLog` is a `Pair<Integer,String>` but `incrWithLog` accepts an `int` as its parameter.

```Java
incrWithLog(absWithLog(-4));  // error
```

We will need to change our methods to take in `Pair<Integer,String>` as the argument.

```Java
Pair<Integer,String> incrWithLog(Pair<Integer,String> p) {
  return Pair.of(incr(p.first), p.second + "; incr " + p.first);
}

Pair<Integer,String> absWithLog(Pair<Integer,String> p) {
  return Pair.of(abs(p.first), p.second + "; abs " + p.first);
}
```

We can now compose the methods.

```Java
incrWithLog(absWithLog(Pair.of(-4, ""))); 
```

## `Loggable` Class

Let's do it in a more OO-way by writing a class to replace `Pair`.

```Java
// version 0.1
class Loggable {
  private final int value;
  private final String log;

  private Loggable(int value, String log) {
    this.value = value;
    this.log = log;
  }

  public static Loggable of(int value) {
    return new Loggable(value, "");
  }

  Loggable incrWithLog() {
    return new Loggable(incr(this.value), this.log + "; incr " + this.value);
  }

  Loggable absWithLog() {
    return new Loggable(abs(this.value), this.log + "; abs " + this.value);
  }

  public String toString() {
    return "value: " + this.value + ", log: " + this.log;
  }
}
```

We can use the class above as:

```Java
Loggable x = Loggable.of(4);
Loggable z = x.incrWithLog().absWithLog();
```

Note that we can now chain the methods together to compose them.  Additionally, the log messages get passed from one call to another and get "composed" as well.

## Making `Loggable` general

There are many possible operations on `int`, and we do not want to add a method `fooWithLog` for every function `foo`.  One way to make `Loggable` general is to abstract out the `int` operation and provide that as a lambda expression to `Loggable`.  This is what the `map` method does. 

```Java
  Loggable map(Transformer<Integer,Integer> transformer) {
    return new Loggable(transformer.transform(this.value), this.log); 
  }
```

We can use it like:

```Java
Loggable.of(4).map(x -> incr(x)).map(x -> abs(x))
```

We can still chain the methods together to compose them.

But, `map` allows us to only apply the function to the value.  What should we do to the log messages?  Since the given lambda returns an int, it is not sufficient to tell us what message we want to add to the log.

To fix this, we will need to pass in a lambda expression that takes in an integer, but return us a pair of integer and a string, in other words, return us a `Loggable`.  We call our new method `flatMap`.

```Java
  Loggable flatMap(Transformer<Integer,Loggable> transformer) {
    Loggable l = transformer.transform(this.value);
    return new Loggable(l.value, l.log + this.log); 
  }
```

By making `flatMap` takes in a lambda that returns a pair of integer and string, `Loggable` can rely on these lambda to tell it how to update the log messages.  Now, if we have methods like this:

```Java
Loggable incrWithLog(int x) {
  return new Loggable(incr(x), "; incr " + x);
}

Loggable absWithLog(int x) {
  return new Loggable(abs(x), "; abs " + x);
}
```

We can write:
```Java
Loggable.of(4)
    .flatMap(x -> incrWithLog(x))
    .flatMap(x -> absWithLog(x))
```

to now compose the methods `incr` and `abs` together, along with the log messages!

## Making `Loggable` More General

We started with an operation on `int`, but our `Loggable` class is fairly general and should be able to add a log message to any operation of any type.  We can make it so by making `Loggable` a generic class.

```Java
// version 0.2
class Loggable<T> {
  private final T value;
  private final String log;

  private Loggable(T value, String log) {
    this.value = value;
    this.log = log;
  }

  public static <T> Loggable<T> of(T value) {
	return new Loggable<>(value, "");
  }

  public <R> Loggable<R> flatMap(Transformer<? super T, ? extends Loggable<? extends R>> transformer) {
    Loggable<? extends R> l = transformer.transform(this.value);
    return new Loggable<>(l.value, l.log + this.log);
  }

  public String toString() {
    return "value: " + this.value + ", log: " + this.log;
  }
}
```

[^1]: This note is inspired by [The Best Introduction to Monad](https://blog.jcoglan.com/2011/03/05/translation-from-haskell-to-javascript-of-selected-portions-of-the-best-introduction-to-monads-ive-ever-read/#). Another excellent notes on category theory is by [Bartosz Milewski](https://bartoszmilewski.com/2014/10/28/category-theory-for-programmers-the-preface/)
