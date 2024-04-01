# Exercise 7: InfiniteList I

- Deadline: 9 April 2024, Tuesday, 23:59, SGT
- Difficulty Level: 9

## Prerequisite

- Caught up to Unit 34 of Lecture Notes
- Completed Exercise 6

## Files

The following functional interfaces are already provided:

- `cs2030s.fp.Combiner`
- `cs2030s.fp.Transformer`
- `cs2030s.fp.BooleanCondition`
- `cs2030s.fp.Producer`
- `cs2030s.fp.Consumer`

Copy your implementation of `cs2030s.fp.Maybe` and `cs2030s.fp.Lazy` over before you start Exercise 7.  A skeleton for `InfiniteList<T>` is provided.

You may add a new method `consumeWith` to `Maybe<T>` to help with solving this lab.
```Java
  /**
   * If the value within this Maybe is missing, do nothing. 
   * Otherwise, consume the value with the given consumer.
   *
   * @param consumer The consumer to consume the value 
   */
  public abstract void consumeWith(Consumer<? super T> consumer);
```

The files `Test1.java`, `Test2.java`, etc., as well as `CS2030STest.java`, are provided for testing.  You can edit them to add your test cases, but they will not be submitted.

## InfiniteList

You have seen in class a poorly implemented version of `InfiniteList`.  Recall that there are two issues: (i) It uses `null` to represent a missing value.  This design prevents us from having `null` as elements in the list; (ii) Produced values are not memoized.  This design results in repeated computation of the same value.

Fortunately, you have built `Maybe<T>` in Lab 5, which will solve (i), and `Lazy<T>` in Lab 6, which will solve (ii).  We will use them to build a better version of `InfiniteList` here.

You are required to implement a single `InfiniteList` class as part of the `cs2030s.fp` package with _only two instance fields.  No other instance fields are needed and allowed_.  You may add one class field (see below).

```Java
public class InfiniteList<T> {
  private Lazy<Maybe<T>> head;
  private Lazy<InfiniteList<T>> tail;
}
```

### Constraints
Take note of the following constraints:

- You are not allowed to use any raw types.
- `@SuppressWarnings` must be used responsibly.
- You must not use `java.util.stream.Stream` to solve this lab.
- You are not allowed to use any conditional statements (e.g., `if`, `else`, `? :`) in your implementation.  Use `Maybe<T>` and `Lazy<T>` to handle the conditional logic.

## The Basics

Write the static `generate` and `iterate` methods that create an `InfiniteList`.

To access the elements of the list, provide the `head` and `tail` method that produces the head and tail of the infinite list.

To help with debugging, a `toString` method has been provided for you.

```Java
jshell> import cs2030s.fp.InfiniteList;
jshell> import cs2030s.fp.Transformer;
jshell> import cs2030s.fp.Producer;

jshell> InfiniteList.generate(() -> 1)
$.. ==> [? ?]
jshell> InfiniteList.generate(() -> 1).head()
$.. ==> 1
jshell> InfiniteList.generate(() -> null).tail().head()
$.. ==> null
jshell> InfiniteList.iterate("A", x -> x + "Z").head()
$.. ==> "A"
jshell> InfiniteList.iterate("A", x -> x + "Z").tail().head()
$.. ==> "AZ"
jshell> InfiniteList.iterate("A", x -> x + "Z").tail().tail().head()
$.. ==> "AZZ"

jshell> Transformer<Integer, Integer> incr = x -> {
   ...>     System.out.println("    iterate: " + x);
   ...>     return x + 1;
   ...> }
jshell> InfiniteList<Integer> numbers = InfiniteList.iterate(1, incr)
jshell> numbers
numbers ==> [[1] ?]

jshell> numbers.head() 
$.. ==> 1
jshell> numbers
numbers ==> [[1] ?]

jshell> numbers.tail().head() 
    iterate: 1
$.. ==> 2
jshell> numbers
numbers ==> [[1] [[2] ?]]

jshell> numbers.tail().head() 
$.. ==> 2
jshell> numbers
numbers ==> [[1] [[2] ?]]

jshell> numbers.tail().tail().head() 
    iterate: 2
$.. ==> 3
jshell> numbers
numbers ==> [[1] [[2] [[3] ?]]]

jshell> numbers.tail().head() 
$.. ==> 2
jshell> numbers
numbers ==> [[1] [[2] [[3] ?]]]

jshell> Producer<Integer> zero = () -> {
   ...>     System.out.println("    generate: 0");
   ...>     return 0;
   ...> }
jshell> InfiniteList<Integer> zeros = InfiniteList.generate(zero)
jshell> zeros
zeros ==> [? ?]

jshell> zeros.head() 
    generate: 0
$.. ==> 0
jshell> zeros
zeros ==> [[0] ?]

jshell> zeros.tail().head() 
    generate: 0
$.. ==> 0
jshell> zeros
zeros ==> [[0] [[0] ?]]

jshell> zeros.head()
$.. ==> 0
jshell> zeros
zeros ==> [[0] [[0] ?]]

jshell> zeros.tail().head()
$.. ==> 0
jshell> zeros
zeros ==> [[0] [[0] ?]]

jshell> zeros.tail().tail().head() 
    generate: 0
$.. ==> 0
jshell> zeros
zeros ==> [[0] [[0] [[0] ?]]]

jshell> zeros.tail().head()
$.. ==> 0
jshell> zeros
zeros ==> [[0] [[0] [[0] ?]]]


```

You can test your code by running the `Test1.java` provided.  The following should compile without errors or warnings.  Make sure your code follows the CS2030S Java style and can generate the documentation without error.
```bash
$ javac -Xlint:rawtypes cs2030s/fp/*java
$ javac -Xlint:rawtypes Test1.java
$ java Test1
$ java -jar ~cs2030s/bin/checkstyle.jar -c ex7_style.xml cs2030s/fp/InfiniteList.java
```

## `map`

Now let's add the `map` method.  The `map` method (lazily) applies the given transformation to each element in the list and returns the resulting `InfiniteList`.

```Java
jshell> import cs2030s.fp.InfiniteList;
jshell> import cs2030s.fp.Transformer;
jshell> import cs2030s.fp.Producer;

jshell> InfiniteList.generate(() -> 1).map(x -> x * 2)
$.. ==> [? ?]
jshell> InfiniteList.iterate(1, x -> x + 1).map(x -> x * 2)
$.. ==> [? ?]
jshell> InfiniteList.generate(() -> 1).map(x -> x * 2).tail().head()
$.. ==> 2
jshell> InfiniteList.iterate(1, x -> x + 1).map(x -> x * 2).head()
$.. ==> 2
jshell> InfiniteList.iterate(1, x -> x + 1).map(x -> x * 2).tail().head()
$.. ==> 4
jshell> InfiniteList.iterate(1, x -> x + 1).map(x -> x * 2).map(x -> x - 1).head()
$.. ==> 1
jshell> InfiniteList.iterate(1, x -> x + 1).map(x -> x * 2).map(x -> x - 1).tail().head()
$.. ==> 3
jshell> InfiniteList.iterate(1, x -> x + 1).map(x -> x % 2 == 0 ? null : x).tail().head()
$.. ==> null

jshell> Producer<Integer> one = () -> {
   ...>     System.out.println("    generate: 1");
   ...>     return 1;
   ...> }
jshell> Transformer<Integer,Integer> doubler = x -> {
   ...>     System.out.println("    map x * 2: " + x);
   ...>     return x * 2;
   ...> }

jshell> InfiniteList.generate(one).map(doubler).tail().head()
    generate: 1
    map x * 2: 1
    generate: 1
    map x * 2: 1
$.. ==> 2

jshell> InfiniteList<Integer> ones = InfiniteList.generate(one)
jshell> InfiniteList<Integer> twos = ones.map(doubler)
jshell> ones
ones ==> [? ?]
jshell> twos
twos ==> [? ?]

jshell> twos.tail().head()
    generate: 1
    map x * 2: 1
    generate: 1
    map x * 2: 1
$.. ==> 2
jshell> ones
ones ==> [[1] [[1] ?]]
jshell> twos
twos ==> [[2] [[2] ?]]

jshell> twos.head()
$.. ==> 2
jshell> twos.tail().head()
$.. ==> 2


```

You can test your code by running the `Test2.java` provided.  The following should compile without errors or warnings.  Make sure your code follows the CS2030S Java style and can generate the documentation without error.

```bash
$ javac -Xlint:rawtypes cs2030s/fp/*java
$ javac -Xlint:rawtypes Test2.java
$ java Test2
$ java -jar ~cs2030s/bin/checkstyle.jar -c ex7_style.xml cs2030s/fp/InfiniteList.java
```

## `filter`

Add the `filter` method to filter out elements in the list that fail a given `BooleanCondition`.  `filter` should mark any filtered (i.e., removed) element as `Maybe.none()` instead of `null`.  The resulting (lazily) filtered `InfiniteList` is returned.

```Java
jshell> import cs2030s.fp.BooleanCondition
jshell> import cs2030s.fp.InfiniteList
jshell> import cs2030s.fp.Transformer

jshell> InfiniteList.generate(() -> 1).filter(x -> x % 2 == 0)
$.. ==> [? ?]
jshell> InfiniteList.iterate(1, x -> x + 1).filter(x -> x % 2 == 0)
$.. ==> [? ?]
jshell> InfiniteList.iterate(1, x -> x + 1).filter(x -> x % 2 == 0).head()
$.. ==> 2
jshell> InfiniteList.iterate(1, x -> x + 1).filter(x -> x % 2 == 0).filter(x -> x > 4).head()
$.. ==> 6

jshell> Transformer<Integer, Integer> incr = x -> {
   ...>   System.out.println("    iterate: " + x);
   ...>   return x + 1;
   ...> }

jshell> BooleanCondition<Integer> isEven = x -> {
   ...>   System.out.println("    filter x % 2 == 0: " + x);
   ...>   return x % 2 == 0;
   ...> }

jshell> InfiniteList.iterate(1, incr).filter(isEven).tail().head()
    filter x % 2 == 0: 1
    iterate: 1
    filter x % 2 == 0: 2
    iterate: 2
    filter x % 2 == 0: 3
    iterate: 3
    filter x % 2 == 0: 4
$.. ==> 4

jshell> InfiniteList<Integer> nums = InfiniteList.iterate(1, x -> x + 1)
jshell> InfiniteList<Integer> evens = nums.filter(x -> x % 2 == 0)

jshell> evens.tail().head()
$.. ==> 4
jshell> nums.toString()
$.. ==> "[[1] [[2] [[3] [[4] ?]]]]"
jshell> evens.toString()
$.. ==> "[[] [[2] [[] [[4] ?]]]]"

jshell> nums.tail().head()
$.. ==> 2
jshell> evens.tail().head()
$.. ==> 4

jshell> BooleanCondition<Integer> moreThan5 = x -> { 
   ...>   System.out.println("    filter x > 5: " + x);
   ...>   return x > 5;
   ...> }
jshell> BooleanCondition<Integer> isEven = x -> { 
   ...>   System.out.println("    filter x % 2 == 0: " + x);
   ...>   return x % 2 == 0;
   ...> }
jshell> Transformer<Integer, Integer> doubler = x -> {
   ...>   System.out.println("    map x * 2: " + x);
   ...>   return x * 2;
   ...> }

jshell> InfiniteList.iterate(1, incr).filter(moreThan5).filter(isEven).head()
    filter x > 5: 1
    iterate: 1
    filter x > 5: 2
    iterate: 2
    filter x > 5: 3
    iterate: 3
    filter x > 5: 4
    iterate: 4
    filter x > 5: 5
    iterate: 5
    filter x > 5: 6
    filter x % 2 == 0: 6
$.. ==> 6
jshell> InfiniteList.iterate(1, incr).map(doubler).filter(moreThan5).filter(isEven).tail().head()
    map x * 2: 1
    filter x > 5: 2
    iterate: 1
    map x * 2: 2
    filter x > 5: 4
    iterate: 2
    map x * 2: 3
    filter x > 5: 6
    filter x % 2 == 0: 6
    iterate: 3
    map x * 2: 4
    filter x > 5: 8
    filter x % 2 == 0: 8
$.. ==> 8
jshell> InfiniteList.iterate(1, incr).filter(isEven).map(doubler).filter(moreThan5).head()
    filter x % 2 == 0: 1
    iterate: 1
    filter x % 2 == 0: 2
    map x * 2: 2
    filter x > 5: 4
    iterate: 2
    filter x % 2 == 0: 3
    iterate: 3
    filter x % 2 == 0: 4
    map x * 2: 4
    filter x > 5: 8
$.. ==> 8


```

You can test your code by running the `Test3.java` provided.  The following should compile without errors or warnings.  Make sure your code follows the CS2030S Java style and can generate the documentation without error.

```Java
$ javac -Xlint:rawtypes cs2030s/fp/*java
$ javac -Xlint:rawtypes Test3.java
$ java Test3
$ java -jar ~cs2030s/bin/checkstyle.jar -c ex7_style.xml cs2030s/fp/InfiniteList.java
```

## Following CS2030S Style Guide

You should make sure that your code follows the [given Java style guide](https://nus-cs2030s.github.io/2324-s2/style.html).

<div class="center">
TO BE CONTINUE
</div>
