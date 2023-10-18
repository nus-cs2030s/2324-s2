# Lab 6: The Art of Being Lazy

!!! abstract "Basic Information"
    - __Deadline:__ 24 October 2023, Tuesday, 23:59 SST
    - __Marks:__ 20
    - __Weightage:__ 2%

!!! info "Prerequisite"
    - Caught up to [Unit 32](../32-lazy.md) of Online Notes.

!!! note "Files"
    In the directory, you should see the following files:

    - __Java Files:__
        - `cs2030s/fp/BooleanCondition.class`: A `BooleanCondition` class.
        - `cs2030s/fp/Consumer.class`: A `Consumer` class.
        - `cs2030s/fp/Producer.class`: A `Producer` class.
        - `cs2030s/fp/Transformer.class`: A `Transfomer` class.
        - `cs2030s/fp/Maybe.class`: A `Maybe` class.
        - `CS2030STest.java`: The main tester class.
        - `Test1.java` - `Test6.java`: A tester.

## Preliminary

This is a follow-up from Lab 5.  In Lab 5, we have constructed a generic class `Maybe<T>`, which is a container for an item of type `T` which may be `null`.  We are now going to use `Maybe<T>` class to construct `Lazy<T>`.

If you have not finished Lab 5, do not worry, we give a possible implementation of Lab 5 as a class file.  Note that this class file was compiled on PE node using Java 17 compiler.  If you are not using Java 17 or if you are not working on the PE node, you may get different result.  Unlikely, but the possibility is there.

### Maybe Class

The class `Maybe<T>` has the following `public` methods.  You cannot use any methods that are not `public`.

| Method | Description |
|--------|-------------|
| `static <T> Maybe<T> of(T val)` | Creates a `Maybe<T>` with the given content `val` if `val` is not `null`.  Otherwise, returns the shared instance of `None<?>`. |
| `String toString()` | Returns the string representation of `Maybe<T>`. |
| `boolean equals(Object obj)` | <ul><li>`Maybe<T>`: Returns `true` if the content is equal to the content of `obj`.  Otherwise returns `false`.</li><li>`None<T>`: Returns `true` if `obj` is also `None<T>`.  Otherwise returns `false`.</li></ul> |
| `<U> Maybe<U> map`<br>`(Transformer<? super T, ? extends U> fn)` | <ul><li>`Maybe<T>`: Create a new instance of `Maybe<T>` by applying the transformer `fn` to the content and wrapping it in `Maybe<T>`. If the result of applying the transformer is `null`, returns `None<T>`.</li><li>`None<T>`: Returns `None<T>`.</li></ul> |
| `Maybe<T> filter`<br>`(BooleanCondition<? super T> pred)` | <ul><li>`Maybe<T>`: If the content is not `null` and `pred.test(content)` returns `true`, we return the current instance.  Otherwise, returns `None<T>`.</li><li>`None<T>`: Returns `None<T>`.</li></ul> |
| `<U> Maybe<U> flatmap`<br>`(Transformer<? super T, ? extends Maybe<? extends U>> fn)` | <ul><li>`Maybe<T>`: Create a new instance of `Maybe<T>` by applying the transformer `fn` to the content _without_ wrapping it in `Maybe<T>` as `fn` already returns `Maybe<U>`.</li><li>`None<T>`: Returns `None<T>`.</li></ul> |
| `T orElse(Producer<? extends T> prod)` | <ul><li>`Maybe<T>`: Returns the content.</li><li>`None<T>`: Returns the value produced by the producer `prod`.</li></ul> |
| `void ifPresent(Consumer<? super T> cons)` | <ul><li>`Maybe<T>`: Pass the content to the consumer `cons`.</li><li>`None<T>`: Do nothing.</li></ul> |

We also have the following `protected` methods that cannot be used outside of the package.

| Method | Description |
|--------|-------------|
| `static <T> Maybe<T> some(T val)` | Creates a `Maybe<T>` with the given content `val` which may be `null`. |
| `static <T> Maybe<T> none()` | Creates a `Maybe<T>` without any content, this is guaranteed to return the shared instance of `None<?>`. |

### The Basics of Being Lazy

Programming languages such as Scala support lazy values, where the expression that produces a lazy value is not evaluated until the value is needed. Lazy value is useful for cases where producing the value is expensive, but the value might not eventually be used. Java, however, does not provide a similar abstraction. So, you are going to build one.

This task is divided into several stages. You are highly encouraged to read through all the stages to see how the different levels are related.

You are required to design a single `Lazy<T>` class as part of the `cs2030s.fp` package with two fields. You are not allowed to add additional fields to `Lazy`.

```java
public class Lazy<T> {
  private Producer<? extends T> producer;
  private Maybe<T> value;

   :
}
```

#### Constraints


## Task 1: ...

To be constructed

### Flexible Type

Make sure you use PECS principle to make the `map` method as flexible as possible.  We have no explicit test case for this.  You are encouraged to make your own test case.


## Grading

This lab is worth 20 marks and contribute 2% to your Lab Assignment component.  The marking scheme is as follows:

| Component | Sub-Component | Marks |
|-----------|---------------|-------|
| Correctness | | 16 marks |
| Style | | 4 marks |

Correctness mark will be about general correctness with more manual checking of types satisfying PECS, doing the proper check (_e.g.,_ `null` _check_), _etc_.  This means that each `TestX.java` may carry different weights.  While we try to give you all the possible tests, there may be some tests we missed.  You should not hardcode any test cases.

Additionally, if your code cannot compile __for any reason__, you will get __0 mark for the lab__.  We will check for compilation with the following command run from `cs2030s/fp` directory.

```bash
javac -Xlint:unchecked -Xlint:rawtypes *.java
```

We may make additional deductions for other issues or errors in your code such as run-time error, failure to follow instructions, failure to correct errors from Lab 5, not commenting `@SuppressWarning`, misuse of `@SuppressWarning` (_unnecessary, not in smallest scope, etc_), etc.

## Submission

To submit the lab, run the following command from the directory containing your lab 6 code.

```sh
~cs2030s/submit-lab6
```

Please check your repo after running the submission script to ensure that your files have been added correctly.  The URL to your repo is given after you run the submission script.

!!! danger "Do NOT Use Other Git Command"
    While you may be familiar with git commands, please do not use them.  Please use only the submission script `submit-labX` to ensure that your submissions are recorded properly.