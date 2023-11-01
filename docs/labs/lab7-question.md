# Lab 7: To Infinity and Beyond

!!! abstract "Basic Information"
    - __Deadline:__ 7 November 2023, Tuesday, 23:59 SST
    - __Marks:__ 20
    - __Weightage:__ 2%

!!! info "Prerequisite"
    - Caught up to [Unit 34](../34-stream.md) of Online Notes.

!!! note "Files"
    In the directory, you should see the following files:

    - __Java Files:__
        - `cs2030s/fp/BooleanCondition.class`: A `BooleanCondition` interface.
        - `cs2030s/fp/Combiner.java`: A `Combiner` interface.
        - `cs2030s/fp/Consumer.class`: A `Consumer` interface.
        - `cs2030s/fp/Producer.class`: A `Producer` interface.
        - `cs2030s/fp/Transformer.class`: A `Transfomer` interface.
        - `cs2030s/fp/Maybe.class` and `cs2030s/fp/Maybe$None.class`: A `Maybe` class.
        - `cs2030s/fp/Lazy.class`: A `Lazy` class.
        - `cs2030s/fp/InfiniteList.java`: A template for `InfiniteList` class.
        - `CS2030STest.java`: The main tester class.
        - `Test1.java` - `Test6.java`: A tester.

## Preliminary

This is a follow-up from Lab 6.  In Lab 6, we have constructed a generic class `Lazy<T>` using `Maybe<T>`.  Now we are going to combine them into a `Lazy<Maybe<T>>` to build an infinite list.  We need the `Lazy<..>` because we want our infinite list to be lazily evaluated.  We need the `Maybe<T>` because the value may be present or may be missing due to `filter`.  Recap that in the lecture notes we use the `null` value to indicate missing values because:

1. We need a value that all possible generic type `T` has.
2. We need a value that indicates a value should not be included.

The only solution was to use `null` because there is no other value that satisfies these two conditions.  Of course, it will be better if we have a _second_ `null` value (_maybe we call it_ `None` _like in Python, heeeyyy wait a minute, that's our_ `None<T>`) to indicate this, but unfortunately we do not have such value.  That is why we need to use `Maybe<T>`.

If you have not finished Lab 6, do not worry, we give a possible implementation of Lab 6 as a class file.  Note that this class file was compiled on PE node using Java 17 compiler.  If you are not using Java 17 or if you are not working on the PE node, you may get different result.  Unlikely, but the possibility is there.

### Maybe Class

The class `Maybe<T>` has the following `public` methods.  You cannot use any methods that are not `public` from outside the package.

| Method | Description |
|--------|-------------|
| `static <T> Maybe<T> of(T val)` | Creates a `Maybe<T>` with the given content `val` if `val` is not `null`.  Otherwise, returns the shared instance of `None<?>`. |
| `static <T> Maybe<T> some(T val)` | Creates a `Maybe<T>` with the given content `val` which may be `null`. |
| `static <T> Maybe<T> none()` | Creates a `Maybe<T>` without any content, this is guaranteed to return the shared instance of `None<?>`. |
| `String toString()` | Returns the string representation of `Maybe<T>`. |
| `boolean equals(Object obj)` | <ul><li>`Maybe<T>`: Returns `true` if the content is equal to the content of `obj`.  Otherwise returns `false`.</li><li>`None<T>`: Returns `true` if `obj` is also `None<T>`.  Otherwise returns `false`.</li></ul> |
| `<U> Maybe<U> map`<br>`(Transformer<? super T, ? extends U> fn)` | <ul><li>`Maybe<T>`: Create a new instance of `Maybe<T>` by applying the transformer `fn` to the content and wrapping it in `Maybe<T>`. ~~If the result of applying the transformer is `null`, returns `None<T>`.~~  ==It will never return `None<T>` to allow for our `InfiniteList<T>` to contain `null`.[^1]==</li><li>`None<T>`: Returns `None<T>`.</li></ul> |
| `Maybe<T> filter`<br>`(BooleanCondition<? super T> pred)` | <ul><li>`Maybe<T>`: If the content is not `null` and `pred.test(content)` returns `true`, we return the current instance.  Otherwise, returns `None<T>`.</li><li>`None<T>`: Returns `None<T>`.</li></ul> |
| `<U> Maybe<U> flatmap`<br>`(Transformer<? super T, ? extends Maybe<? extends U>> fn)` | <ul><li>`Maybe<T>`: Create a new instance of `Maybe<T>` by applying the transformer `fn` to the content _without_ wrapping it in `Maybe<T>` as `fn` already returns `Maybe<U>`.</li><li>`None<T>`: Returns `None<T>`.</li></ul> |
| `T orElse(Producer<? extends T> prod)` | <ul><li>`Maybe<T>`: Returns the content (_even if it is_ `null`).</li><li>`None<T>`: Returns the value produced by the producer `prod`.</li></ul> |
| `void ifPresent(Consumer<? super T> cons)` | <ul><li>`Maybe<T>`: Pass the content to the consumer `cons`.</li><li>`None<T>`: Do nothing.</li></ul> |

[^1]: Do not worry too much about this, we have already implemented `map` and `filter` for you.

### Lazy Class

The class `Lazy<T>` has the following `public` methods.  You cannot use any methods that are not `public` from outside the package.

| Method | Description |
|--------|-------------|
| `static <T> Lazy<T> of(T val)` | Creates a `Lazy<T>` with the given content `val` already evaluated. |
| `static <T> Lazy<T> of(Producer<? extends T> prod)` | Creates a `Lazy<T>` with the content not yet evaluated and will be evaluated using the given producer. |
| `boolean equals(Object obj)` | Returns `true` if the content is equal to the content of `obj`.  Otherwise returns `false`.  This __forces__ evaluation of the content. |
| `<U> Lazy<U> map`<br>`(Transformer<? super T, ? extends U> fn)` | Lazily maps the content using the given transformer. |
| `Lazy<Boolean> filter`<br>`(BooleanCondition<? super T> pred)` | Lazily test if the value passes the test or not and returns a `Lazy<Boolean>` to indicate the result. |
| `<U> Lazy<U> flatmap`<br>`(Transformer<? super T, ? extends Lazy<? extends U>> fn)` | Lazily creates a new instance of `Lazy<T>` by applying the transformer `fn` to the content without wrapping it in another `Lazy<..>`. |
| `T get()` | Evaluates (_if not yet evaluated, otherwise do not evaluate again_) and returns the content. |

### Infinity

You have seen in class a poorly implemented version of `InfiniteList`. Recall that there are two issues: (i) It uses `null` to represent a missing value. This design prevents us from having `null` as elements in the list; (ii) Produced values are not memoized. This design results in repeated computation of the same value.

Fortunately, you have built `Maybe<T>` in Lab 5, which will solve (i), and `Lazy<T>` in Lab 6, which will solve (ii). We will use them to build a better version of `InfiniteList` here.

You are required to implement a single `InfiniteList` class as part of the `cs2030s.fp` package with only two instance fields. No other instance fields are needed and allowed. There is also a single class field.

```java
public class InfiniteList<T> {
  private final Lazy<Maybe<T>> head;
  private final Lazy<InfiniteList<T>> tail;
  private static final InfiniteList<?> SENTINEL = (InfiniteList<?>) new Sentinel();
}
```

Please take note of the following constraints.

!!! danger "Constraints"
    We will recap some of the constraints for our labs so far.

    - You are __NOT__ allowed to use any raw types.
    - `@SuppressWarnings` must be used responsibly, in the smallest scope, and commented.

    Additionally, you have the following constraints for this lab.

    - You are __NOT__ allowed to use `java.util.stream.Stream`.
    - You are __NOT__ allowed to add new classes (_nested or otherwise_).
    - You are __NOT__ allowed to add new methods in the `InfiniteList` (_not even private methods_).
    - You are __NOT__ allowed to use conditional statement (_e.g.,_ `if`-`else`) or conditional expression (_e.g.,_ `?:` _operator_).
        - Unless otherwise stated.
    - You are __NOT__ allowed to use loops (_e.g., while-loop or for-loop_).
        - You may, however, use _recursion_ (_but possibly without conditionals_).
    - There must only be a single __instance__ of `Sentinel`.

However, as a relaxation, you are already given the most flexible type.

!!! info "Flexible Type"
    The type signature in the templates are already the most flexible types.  You have suffered enough thinking about more flexible type in the past two labs.  The focus here is about laziness.

### The Basics

You are already given most of the implementations including `head()`, `tail()`, `map(..)`, and `filter(..)`.  Please study them carefully.  Additionally, to help with debugging, a `toString()` has already been provided for you.  Lastly, there are three factory methods for `InfiniteList`, namely `generate(..)`, `iterate(..)`, and `sentinel()`.  The last one creates an empty list.

As we also want to limit our infinite list to a potentially finite list, we have provided the `Sentinel` class.  This class is rather straightforward as it overrides all of the methods in `InfiniteList<T>`.  However, in most cases, it simply returns itself (_through the use of_ `InfiniteList.<Object>sentinel()`) or throw an exception.

You can test the initial implementation by running `Test1.java` to `Test3.java`.

```
$ javac -Xlint:rawtypes cs2030s/fp/*java
$ javac -Xlint:rawtypes Test1.java
$ javac -Xlint:rawtypes Test2.java
$ javac -Xlint:rawtypes Test3.java
$ java Test1
$ java Test2
$ java Test3
```

### Documenting Your Code

Now that we are beginning to build our own package that others can use, we should start to produce documentation on our code.

From Lab 7 onwards, you are required to document your classes and methods with Javadoc comments. You have seen examples from the skeleton code in Lab 1 given earlier. For more details, see the [JavaDoc](../javadoc.md) guide. The checkstyle tool now checks for JavaDoc-related style as well.

For Lab 7, you should write javadoc documentation for all methods in `InfiniteList.java`. Documenting the code your wrote previously for Lab 5 is encouraged but optional.

## Task 1: Document the Remaining Methods

First order of business is to document the remaining methods.  We have provided some documentations on some of the codes as example.  You should also double-check that the provided documentations satisfies the style guide.

```
$ javac cs2030s/fp/InfiniteList.java
$ java -jar ~cs2030s/bin/checkstyle.jar -c ~cs2030s/bin/cs2030_checks.xml cs2030s/fp/InfiniteList.java
$ javadoc -quiet -private -d docs cs2030s/fp/InfiniteList.java
```

## Task 2: Limit and ToList

The `Sentinel` class is not only an indication of an empty list but because our idea of an `InfiniteList` is the value (_or value wrapped in_ `Lazy` _and_ `Maybe`, _i.e., the head_) and the rest of the list (_which is another_ `InfiniteList`), an empty list is also an indicator for the end of the list.  Given a `Sentinel`, we can now write two methods:

1. Implement the method `InfiniteList<T> limit(long n)` in `InfiniteList`.
    - The method takes in a number `n`.
    - The method returns a new `InfiniteList` that is lazily computed which is the _truncation_ of the `InfiniteList` to a finite list with _at most_ `n` elements.
    - The method should not count the elements that are filtered out by `filter` (_if any_).
    - The method is __allowed__ to use conditional statement/expression.
2. Override the method `InfiniteList<T> limit(long n)` in `Sentinel`.
    - The method takes in a number `n`.
    - Determine the appropriate behaviour for this.
3. Implement the method `List<T> toList()` in `InfiniteList`.
    - The method takes in no parameter.
    - The method returns a new [`List<T>`](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/List.html) (_should really just be_ [`ArrayList<T>`](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/ArrayList.html)) which is a collection elements of the `InfiniteList` in the same order as they appear in the `InfiniteList`.
4. Override the method `InfiniteList<T> toList()` in `Sentinel`.
    - The method takes in no parameter.
    - Determine the appropriate behaviour for this.

### Sample Usage

```
jshell> import cs2030s.fp.BooleanCondition
jshell> import cs2030s.fp.InfiniteList
jshell> import cs2030s.fp.Transformer
jshell> import cs2030s.fp.Producer

jshell> InfiniteList.sentinel().limit(4).isSentinel()
$.. ==> true
jshell> InfiniteList.iterate(1, x -> x + 1).limit(0).isSentinel()
$.. ==> true
jshell> InfiniteList.iterate(1, x -> x + 1).limit(1).isSentinel()
$.. ==> false
jshell> InfiniteList.iterate(1, x -> x + 1).limit(10).isSentinel()
$.. ==> false
jshell> InfiniteList.iterate(1, x -> x + 1).limit(-1).isSentinel()
$.. ==> true
jshell> InfiniteList.iterate(1, x -> x + 1).limit(0).isSentinel()
$.. ==> true
jshell> InfiniteList.iterate(1, x -> x + 1).limit(1).isSentinel()
$.. ==> false
jshell> InfiniteList.iterate(1, x -> x + 1).limit(10).isSentinel()
$.. ==> false

jshell> InfiniteList.generate(() -> 1).limit(4)
$.. ==> [? ?]
jshell> InfiniteList.iterate(1, x -> x + 1).limit(4)
$.. ==> [[1] ?]
jshell> InfiniteList.iterate(1, x -> x + 1).limit(1).head()
$.. ==> 1
jshell> InfiniteList.iterate(1, x -> x + 1).limit(4).head()
$.. ==> 1

jshell> <T> T run(Producer<T> p) {
   ...>   try {
   ...>     return p.produce();
   ...>   } catch (Exception e) {
   ...>     System.out.println(e);
   ...>     return null;
   ...>   }
   ...> }

jshell> run(() -> InfiniteList.iterate(1, x -> x + 1).limit(1).tail().head());
java.util.NoSuchElementException
$.. ==> null
jshell> run(() -> InfiniteList.iterate(1, x -> x + 1).limit(0).head()); 
java.util.NoSuchElementException
$.. ==> null
jshell> run(() -> InfiniteList.iterate(1, x -> x + 1).limit(4).tail().tail().head());
$.. ==> 3
jshell> run(() -> InfiniteList.iterate(1, x -> x + 1).limit(4).limit(1).tail().head());
java.util.NoSuchElementException
$.. ==> null
jshell> run(() -> InfiniteList.iterate(1, x -> x + 1).limit(1).limit(4).tail().head());
java.util.NoSuchElementException
$.. ==> null

jshell> run(() -> InfiniteList.iterate(1, x -> x + 1).filter(x -> x % 2 == 0).limit(0).head());
java.util.NoSuchElementException
$.. ==> null
jshell> run(() -> InfiniteList.iterate(1, x -> x + 1).filter(x -> x % 2 == 0).limit(1).head());
$.. ==> 2
jshell> run(() -> InfiniteList.iterate(1, x -> x + 1).limit(1).filter(x -> x % 2 == 0).head());
java.util.NoSuchElementException
$.. ==> null
jshell> run(() -> InfiniteList.iterate(1, x -> x + 1).limit(2).filter(x -> x % 2 == 0).head());
$.. ==> 2

jshell> run(() -> InfiniteList.iterate("A", s -> s + "Z").limit(2).map(s -> s.length()).head());
$.. ==> 1
jshell> run(() -> InfiniteList.iterate("A", s -> s + "Z").limit(2).map(s -> s.length()).tail().head());
$.. ==> 2
jshell> run(() -> InfiniteList.iterate("A", s -> s + "Z").limit(2).map(s -> s.length()).tail().tail().head());
java.util.NoSuchElementException
$.. ==> null

jshell> run(() -> InfiniteList.iterate("A", s -> s + "Z").map(s -> s.length()).limit(2).head());
$.. ==> 1
jshell> run(() -> InfiniteList.iterate("A", s -> s + "Z").map(s -> s.length()).limit(2).tail().head());
$.. ==> 2
jshell> run(() -> InfiniteList.iterate("A", s -> s + "Z").map(s -> s.length()).limit(2).tail().tail().head());
java.util.NoSuchElementException
$.. ==> null

jshell> InfiniteList.<String>sentinel().toList()
$.. ==> []
jshell> InfiniteList.iterate("A", s -> s + "Z").map(s -> s.length()).limit(2).toList()
$.. ==> [1, 2]
jshell> InfiniteList.iterate("A", s -> s + "Z").limit(2).map(s -> s.length()).toList()
$.. ==> [1, 2]
jshell> InfiniteList.iterate(1, x -> x + 1).limit(2).filter(x -> x % 2 == 0).toList()
$.. ==> [2]
jshell> InfiniteList.iterate(1, x -> x + 1).filter(x -> x % 2 == 0).limit(2).toList()
$.. ==> [2, 4]
jshell> InfiniteList.iterate(0, x -> x + 1).filter(x -> x > 10).map(x -> x.hashCode() % 30).filter(x -> x < 20).limit(5).toList()
$.. ==> [11, 12, 13, 14, 15]
jshell> Random rng = new Random(1)
jshell> InfiniteList.generate(() -> rng.nextInt() % 100).filter(x -> x > 10).limit(4).toList()
$.. ==> [76, 95, 26, 69]
jshell> InfiniteList.generate(() -> null).limit(4).limit(1).toList()
$.. ==> [null]
jshell> InfiniteList.generate(() -> null).limit(1).limit(4).toList()
$.. ==> [null]
```

You can test your code by running the `Test4.java` provided. The following should compile without errors or warnings. Make sure your code follows the CS2030S Java style. 

```
$ javac -Xlint:rawtypes cs2030s/fp/*java
$ javac -Xlint:rawtypes Test4.java
$ java Test4
$ java -jar ~cs2030s/bin/checkstyle.jar -c ~cs2030s/bin/cs2030_checks.xml cs2030s/fp/InfiniteList.java
$ javadoc -quiet -private -d docs cs2030s/fp/InfiniteList.java
```

## Task 3: It Takes a While

Now we want to implement the `takeWhile` method.

1. Implement the method `InfiniteList<T> takeWhile(BooleanCondition<? super T> pred)` in `InfiniteList`.
    - The method takes in a `BooleanCondition`.
    - The method returns an `InfiniteList` that is a truncated version of the initial `InfiniteList`.
        - The method truncates the infinite list as soon as it finds an element that evaluates the condition to `false`.
    - The method is __allowed__ to use conditional statement/expression.
2. Override the method `InfiniteList<T> takeWhile(BooleanCondition<? super T> pred)` in `Sentinel`.
    - The method takes in a `BooleanCondition`.
    - Determine the appropriate behaviour for this.

### Sample Usage

```
jshell> import cs2030s.fp.InfiniteList;
jshell> import cs2030s.fp.Transformer;
jshell> import cs2030s.fp.Producer;
jshell> import cs2030s.fp.BooleanCondition;

jshell> Transformer<Integer, Integer> incr = x -> { 
   ...>   System.out.println("    iterate: " + x);
   ...>   return x + 1;
   ...> };
jshell> BooleanCondition<Integer> lessThan0 = x -> { 
   ...>   System.out.println("    takeWhile x < 0: " + x);
   ...>   return x < 0;
   ...> };
jshell> BooleanCondition<Integer> lessThan2 = x -> { 
   ...>   System.out.println("    takeWhile x < 2: " + x);
   ...>   return x < 2;
   ...> };
jshell> BooleanCondition<Integer> lessThan5 = x -> { 
   ...>   System.out.println("    takeWhile x < 5: " + x);
   ...>   return x < 5;
   ...> };
jshell> BooleanCondition<Integer> lessThan10 = x -> { 
   ...>   System.out.println("    takeWhile x < 10: " + x);
   ...>   return x < 10;
   ...> };
jshell> BooleanCondition<Integer> isEven = x -> { 
   ...>   System.out.println("    filter x % 2 == 0: " + x);
   ...>   return x % 2 == 0;
   ...> };

jshell> <T> T run(Producer<T> p) {
   ...>   try {
   ...>     return p.produce();
   ...>   } catch (Exception e) {
   ...>     System.out.println(e);
   ...>     return null;
   ...>   }
   ...> }

jshell> InfiniteList.<Integer>sentinel().takeWhile(lessThan0).isSentinel()
$.. ==> true
jshell> InfiniteList.iterate(1, incr).takeWhile(lessThan0).isSentinel()
$.. ==> false
jshell> InfiniteList.iterate(1, incr).takeWhile(lessThan2).isSentinel()
$.. ==> false
jshell> InfiniteList.iterate(1, incr).takeWhile(lessThan5).takeWhile(lessThan2).toList()
    takeWhile x < 5: 1
    takeWhile x < 2: 1
    iterate: 1
    takeWhile x < 5: 2
    takeWhile x < 2: 2
$.. ==> [1]
jshell> InfiniteList.iterate(1, incr).filter(isEven).takeWhile(lessThan10).toList()
    filter x % 2 == 0: 1
    iterate: 1
    filter x % 2 == 0: 2
    takeWhile x < 10: 2
    iterate: 2
    filter x % 2 == 0: 3
    iterate: 3
    filter x % 2 == 0: 4
    takeWhile x < 10: 4
    iterate: 4
    filter x % 2 == 0: 5
    iterate: 5
    filter x % 2 == 0: 6
    takeWhile x < 10: 6
    iterate: 6
    filter x % 2 == 0: 7
    iterate: 7
    filter x % 2 == 0: 8
    takeWhile x < 10: 8
    iterate: 8
    filter x % 2 == 0: 9
    iterate: 9
    filter x % 2 == 0: 10
    takeWhile x < 10: 10
$.. ==> [2, 4, 6, 8]

jshell> run(() -> InfiniteList.generate(() -> 2).takeWhile(lessThan0));
$.. ==> [? ?]
jshell> run(() -> InfiniteList.iterate(1, incr).takeWhile(lessThan0));
$.. ==> [? ?]
jshell> run(() -> InfiniteList.iterate(1, incr).takeWhile(lessThan0).head());
    takeWhile x < 0: 1
java.util.NoSuchElementException
$.. ==> null
jshell> run(() -> InfiniteList.iterate(1, incr).takeWhile(lessThan2).head());
    takeWhile x < 2: 1
$.. ==> 1
jshell> run(() -> InfiniteList.iterate(1, incr).takeWhile(lessThan2).tail().head());
    takeWhile x < 2: 1
    iterate: 1
    takeWhile x < 2: 2
java.util.NoSuchElementException
$.. ==> null
jshell> run(() -> InfiniteList.iterate(1, incr).takeWhile(lessThan2).takeWhile(lessThan0).head());
    takeWhile x < 2: 1
    takeWhile x < 0: 1
java.util.NoSuchElementException
$.. ==> null
jshell> run(() -> InfiniteList.iterate(1, incr).takeWhile(lessThan0).takeWhile(lessThan2).head());
    takeWhile x < 0: 1
java.util.NoSuchElementException
$.. ==> null
jshell> run(() -> InfiniteList.iterate(1, incr).takeWhile(lessThan5).takeWhile(lessThan2).tail().head());
    takeWhile x < 5: 1
    takeWhile x < 2: 1
    iterate: 1
    takeWhile x < 5: 2
    takeWhile x < 2: 2
java.util.NoSuchElementException
$.. ==> null
jshell> run(() -> InfiniteList.iterate(1, incr).filter(isEven).takeWhile(lessThan10).head());
    filter x % 2 == 0: 1
    iterate: 1
    filter x % 2 == 0: 2
    takeWhile x < 10: 2
$.. ==> 2
jshell> run(() -> InfiniteList.iterate(1, incr).filter(isEven).takeWhile(lessThan10).tail().head());
    filter x % 2 == 0: 1
    iterate: 1
    filter x % 2 == 0: 2
    takeWhile x < 10: 2
    iterate: 2
    filter x % 2 == 0: 3
    iterate: 3
    filter x % 2 == 0: 4
    takeWhile x < 10: 4
$.. ==> 4

jshell> InfiniteList<Integer> list = InfiniteList.iterate(1, incr).takeWhile(lessThan10)

jshell> list.tail().tail().head()
    takeWhile x < 10: 1
    iterate: 1
    takeWhile x < 10: 2
    iterate: 2
    takeWhile x < 10: 3
$.. ==> 3
jshell> list.head()
$.. ==> 1
jshell> list
list ==> [[1] [[2] [[3] ?]]]

jshell> list.tail().head()
$.. ==> 2
jshell> list.tail().tail().tail().head()
    iterate: 3
    takeWhile x < 10: 4
$.. ==> 4
jshell> list
list ==> [[1] [[2] [[3] [[4] ?]]]]
```

You can test your code by running the `Test5.java` provided. The following should compile without errors or warnings. Make sure your code follows the CS2030S Java style. 

```
$ javac -Xlint:rawtypes cs2030s/fp/*java
$ javac -Xlint:rawtypes Test5.java
$ java Test5
$ java -jar ~cs2030s/bin/checkstyle.jar -c ~cs2030s/bin/cs2030_checks.xml cs2030s/fp/InfiniteList.java
$ javadoc -quiet -private -d docs cs2030s/fp/InfiniteList.java
```

## Task 4: Folding it Right

In the lecture, we discuss about the behaviour of `reduce` method.  We mentioned that it is equivalent to fold left.  In a fold left, given a list `[v1, v2, v3, v4]`, an initial value `e` and the operation `+`, the operation performed is as follows:

`(((e + v1) + v2) + v3) + v4`

What we want is the opposite.  We want to do a fold but from the right.  In a fold left, given a list `[v1, v2, v3, v4]`, an initial value `e` and the operation `+`, the operation performed is as follows:

`(v1 + (v2 + (v3 + (v4 + e))))`

Since the order of operations are different, the result may potentially be different.  For `+`, these two are going to produce the same result.  But for `-`, the result will be different.  Note that this is a _terminal operation_ in Java stream.

1. Implement the method `<U> U foldRight(U id, Combiner<? super T, U, U> acc)` in `InfiniteList`.
    - The method takes in the initial value `id` and an accumulator `acc`.
        - Note that `Combiner<T, S, R>` is a new interface with the implementation given in `cs2030s/fp/Combiner.java`.
        - It has a single abstract method `R combine(T arg1, S arg2)`.
    - The method returns the result of fold right of type `U` performed on the `InfiniteList` with the given accumulator `acc` starting from the initial value `id`.
        - Note that the name `id` is used as typically the initial value is the _identity_ operation of the accumulator.
2. Override the method `<U> U foldRight(U id, Combiner<? super T, U, U> acc)` in `Sentinel`.
    - The method takes in the initial value `id` and an accumulator `acc`.
    - Determine the appropriate behaviour for this.

### Sample Usage

```
jshell> import cs2030s.fp.InfiniteList;

jshell> InfiniteList.<Integer>sentinel().foldRight(0, (x, y) -> x + y)
$.. ==> 0
jshell> InfiniteList.iterate(0, x -> x + 1).limit(5).foldRight(0, (x, y) -> x + y)
$.. ==> 10
jshell> InfiniteList.iterate(0, x -> x + 1).limit(0).foldRight(0, (x, y) -> x + y)
$.. ==> 0
jshell> InfiniteList.iterate(1, x -> x + 1).map(x -> x * x).limit(5).foldRight(1, (x, y) -> x * y)
$.. ==> 14400
jshell> InfiniteList.iterate(1, x -> x + 1).map(x -> x * x).limit(5).foldRight(1, (x, y) -> x - y)
$.. ==> 14
jshell> // the above is equivalent to (1 - (4 - (9 - (16 - (25 - 1)))))
```

You can test your code by running the `Test6.java` provided. The following should compile without errors or warnings. Make sure your code follows the CS2030S Java style. 

```
$ javac -Xlint:rawtypes cs2030s/fp/*java
$ javac -Xlint:rawtypes Test6.java
$ java Test6
$ java -jar ~cs2030s/bin/checkstyle.jar -c ~cs2030s/bin/cs2030_checks.xml cs2030s/fp/InfiniteList.java
$ javadoc -quiet -private -d docs cs2030s/fp/InfiniteList.java
```

## Grading

This lab is worth 20 marks and contribute 2% to your Lab Assignment component.  The marking scheme is as follows:

| Component | Marks | Comments |
|-----------|---------------|-------|
| Correctness | 16 marks |  |
| Documentation | 4 marks |  |
| Style | _up to -4 marks_ | _Negative Marking_ |

Correctness mark will be about general correctness with some manual checking, doing the proper check (_e.g.,_ `null` _check_), laziness, not using conditional statement/expression, not using loops, _etc_.  This means that each `TestX.java` may carry different weights.  While we try to give you all the possible tests, there may be some tests we missed.  You should not hardcode any test cases.

Additionally, if your code cannot compile __for any reason__, you will get __0 mark for the lab__.  We will check for compilation with the following command.

```bash
javac -Xlint:unchecked -Xlint:rawtypes cs2030s/fp/*.java
```

We may make additional deductions for other issues or errors in your code such as run-time error, failure to follow instructions, failure to correct errors from Lab 5, not commenting `@SuppressWarning`, misuse of `@SuppressWarning` (_unnecessary, not in smallest scope, etc_), having unnecessary conditional statements/expressions, etc.

Note that style marks are no longer awarded. You should know how to follow the prescribed Java style by now. We will deduct up to 4 marks if there are serious violations of styles.

## Submission

To submit the lab, run the following command from the directory containing your lab 7 code.

```sh
~cs2030s/submit-lab7
```

Please check your repo after running the submission script to ensure that your files have been added correctly.  The URL to your repo is given after you run the submission script.

!!! danger "Do NOT Use Other Git Command"
    While you may be familiar with git commands, please do not use them.  Please use only the submission script `submit-labX` to ensure that your submissions are recorded properly.