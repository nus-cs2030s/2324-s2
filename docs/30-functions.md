# Unit 30: Side Effect-Free Programming

!!! abstract "Learning Objectives"

    Students should

    - appreciate the concept of functions as side-effect-free programming constructs and its relation to functions in mathematics.
    - understand the importance of writing code that is free of side effects.
    - understand how functions can be first-class citizens in Java through using local anonymous class.
    - understand how we can succinctly use a lambda expression or a method reference in place of using local anonymous class.
    - understand how we can use currying to generalize to functions with higher arity.
    - understand how we can create a closure with lambda and environment capture.

## Functions

Recall that, a function, in mathematics, refers to a mapping from a set of inputs (_domain_) $X$ to a set of output values (_codomain_) $Y$.  We write $f: X \rightarrow Y$.  Every input in the domain must map to exactly one output but multiple inputs can map to the same output.  Not all values in the codomain need to be mapped.

Let us illustrate this.  We can represent the `abs` function as a mapping from `int` to `int`.  So the domain is `int` and the codomain is `int`.  If the value is negative, we map the value to its positive value.  On the other hand, if the value is already positive, we map the value to itself.  This shows that not all values in the codomain are mapped.  However, all values in the domain must be mapped and, in fact, they are mapped to exactly one value in the codomain.

![abs](figures/Functions.png)

We know how to deal with mathematical functions very well.  There are certain rules that we follow when we reason about functions.  For instance, suppose we have an unknown $x$ and a function $f$, we know that applying $f$ on $x$, i.e., $f(x)$ does not change the value of $x$, or any other unknowns $y$, $z$, etc.  We say that mathematical functions have _no side effects_.  It simply computes and returns the value.

!!! info "Why Side-Effect?"
    The term side-effect should be compared with the main-effect (_or primary effect_) of a function.  Mathematically, a function is a transformation from one value to another value.  So the main-effect is to return that value without modifying any of the input.  Anything else is a side-effect.  Side-effect includes:

    - Printing to the screen or other I/O write operation.
    - Modify the value of a field.
    - Mutating the input arguments.
    - Invoking other functions with side-effects.
    - Throwing exceptions.
    - _etc_.

    Also note that observable behavior caused by the time taken for an operation to execute are ignored.

Another property of mathematical function is _referential transparency_.  Let $f(x) = a$.  Then in every formula that $f(x)$ appears in, we can safely replace occurences of $f(x)$ with $a$.  Conversely, everywhere $a$ appears, we can replace it with $f(x)$.  We can be guarantee that the resulting formulas are still equivalent.

Note that absence of side-effect does not guarantee referential transparency.  The function that returns the current time has no side-effect (_i.e., it does not change the time, does not print on the screen, etc_) but it is still not referentially transparent.  Invoking the same function at two different time produces different values.  To be referentially transparent, a function must also be deterministic.

These two fundamental properties of mathematical functions allow us to solve equations, prove theorems, and reason about mathematical objects rigorously.  We call functions that are both side-effect free and referentially transparent as ___pure functions___.

Unfortunately, we can't always reason about our program the same way as we reason about mathematical functions.  For instance, consider the line:

```java
a.get(0)
```

where `a` is an instance of `Array<T>`.  Suppose we know that `a.get(0)` is 5 for some `a`.  When we reason about the behavior of our code, we cannot replace (_mentally_) every invocation of `a.get(0)` with the value 5.  This is because the array `a` may not be immutable and therefore `a.get(0)` cannot be guaranteed to be the same.

=== "Can `a.get(0)` be Replaced with 5?"
    ```java
    a.get(0); // 5
      :
    a.get(0); // is it 5?
    ```
=== "No!"
    ```java
    a.get(0); // 5
    a.set(0, 10);
    a.get(0); // is it 5? NO, it's 10!
    ```

The reverse should be true as well.  Suppose we have a variable

```java
T t = a.get(0);
```

Then everywhere in our code where we use `t`, we should be able to replace it with `a.get(0)`, and the behavior of the code should still be the same. This behavior is only guaranteed if `a.get(0)` has no side effects (_such as modifying a field or print something to the standard output_).

Since our `Array<T>` code is written rather well, we will illustrate it with the following example.

```java
class Counter {
  private int count;
  private int value;

  public Counter(int value) {
    this.value = value;
  }

  public int getValue() {
    this.count++;
    return this.value;
  }

  public int getCount() {
    return this.count;
  }
}
```

Now suppose we have the following variable.

```java
Counter ctx = new Counter(10);
int val = ctx.getValue();
```

The question now is can we replace `val` with `ctx.getValue()` everywhere?  See the effect below.

=== "Using `val`"
    ```java hl_lines="3-5"
    Counter ctx = new Counter(10);
    int val = ctx.getValue();
    int tmp1 = val;
    int tmp2 = val;
    int tmp3 = val;
    int end = tmp1 + tmp2 + tmp3 + ctx.getCount(); // 31
    ```
=== "Using `ctx.getValue()`"
    ```java hl_lines="3-5"
    Counter ctx = new Counter(10);
    int val = ctx.getValue();
    int tmp1 = ctx.getValue();
    int tmp2 = ctx.getValue();
    int tmp3 = ctx.getValue();
    int end = tmp1 + tmp2 + tmp3 + ctx.getCount(); // 34
    ```

To be able to reason about our code using the mathematical reasoning techniques we are familiar with, it is important to write our code as if we are writing mathematical functions -- our methods should be free of side effects and our code should be referentially transparent.  Our program is then just a sequence of functions, chained and composed together.  To achieve this, functions need to be a _first class citizen_ in our program, so that we can assign functions to a variable, pass it as parameters, return a function from another function, etc, just like any other variable.

## Pure Functions

Ideally, methods in our programs should behave the same as functions in mathematics.  Given an input, the function computes and returns an output.  A _pure_ function does nothing else -- it does not print to the screen, write to files, throw exceptions, change other variables, modify the values of the arguments, etc.  That is, a pure function does not cause any _side effect_ and is _referentially transparent_.

Here are two examples of pure functions:

```Java
int square(int i) {
  return i * i;
}

int add(int i, int j) {
  return i + j;  // believe it or not, an overflow is
                 // not an error in Java
}
```

and some examples of non-pure functions:
```Java
int div(int i, int j) {
  return i / j;  // may throw an exception
}

int incrCount(int i) {
  return this.count + i; // assume that count is not final.
                         // this may give diff results for the same i.
}

void incrCount(int i) {
  this.count += i; // does not return a value
                   // and has side effects on count
}

int addToQueue(Queue<Integer> queue, int i) {
  queue.enq(i);  // has side effects on queue
}
```

A pure function must also be deterministic.  Given the same input, the function must produce the same output, _every single time_.  As we said before, this deterministic property ensures referential transparency.

In the OO paradigm, we commonly need to write methods that update the fields of an instance or compute values using the fields of an instance.  Such methods are not pure functions.  On the other hand, if our class is immutable, then its methods must not have side effects and thus is pure.

In computer science, we refer to the style of programming where we build a program from pure functions as _functional programming_ (FP). Examples of functional programming languages include Haskell, OCaml, Erlang, Clojure, F#, and Elixir.

Many modern programming languages including Java, C++, Python, Rust, and Swift support this style of programming.  As these languages are not designed to be functional, we cannot build a program from only pure functions.  Java, for instance, is still an OO language at its core.  As such, we will refer to this style as _functional-style programming_.  We won't be able to write code consists of only pure functions in Java, but we can write methods that has no side effects and objects that are immutable, as much as possible.

## Function as First-Class Citizen in Java

Let's explore functions as a first-class citizen in Java.  We have seen some examples of this when we use the `Comparator` interface.

```Java
void sortNames(List<String> names) {
  Comparator<String> cmp = new Comparator<String>() {
    public int compare(String s1, String s2) {
    return s1.length() - s2.length();
    }
  };
  names.sort(cmp);
}
```

First, let's take a moment to appreciate the beauty of the `List::sort` method.  We can use this method to sort items of _any type_, in _any defined order_.  We achieve the generality of types with generics, and the generality of sorting order through passing in the comparison function as a parameter.  The latter is needed to write one sorting method for every possible sorting order for a list of strings, (`sortAlphabeticallyIncreasing`, `sortByLengthDecreasing`, etc..)

The comparison function here is implemented _as a method in an anonymous class that implements an interface_.  We can think of an instance of this anonymous class as the function.  Since a function is now just an instance of an object in Java, we can pass it around, return it from a function, and assign it to a variable, just like any other reference type.

Let's look at another example.  Consider the following interface:
```Java
interface Transformer<T, R> {
  R transform(T t);
}
```

`Transformer<T, R>` is a generic interface with two type parameters: `T` is the type of the input, `R` is the type of the result.  It has one abstract method `R transform(T t)` that applies the function to a given argument.

We can use this interface to write any function that takes in a value and return another value.  (_Java has a similar interface called, unsurprisingly,_ [`java.util.function.Function<T, R>`](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/Function.html)). For instance, a function that computes the square of an integer can be written as:
```Java
new Transformer<Integer, Integer>() {
  @Override
  public Integer transform(Integer x) {
    return x * x;
  }
};
```

We can write a method `chain` that composes two given computations together and
return the new computation:
```Java
// Use of PECS left as an exercise to the reader
<T, R, S> Transformer<T,R> chain(Transformer<T,S> t1, Transformer<S,R> t2) {
  return new Transformer<T,R>() {
    public R transform(T value) {
      return t2.transform(t1.transform(value));
      // in mathematical notation, this is t2(t1(v))
    }
  }
}
```

Note that the use of local class above is necessary in the method `chain` above.  This is because we need to be able to invoke both `t1` and `t2`.  Since this local class is only used to connect both `t1` and `t2`, we do not need to give it a name.  Hence, anonymous class is suitable for this.

## Lambda Expression

While we have achieved functions as first-class citizens in Java, the code is verbose and ugly.  Fortunately, there is a much cleaner syntax to write functions that applies to interfaces with a single abstract method.

An interface in Java with __exactly__ one abstract method (_either declared in the interface or inherited_) is called a _functional interface_.  Both `Comparator` and `Transformer` are functional interfaces.  It is recommended that, if a programmer intends an interface to be a functional interface, they should annotate the interface with the `@FunctionalInterface` annotation.

```Java
@FunctionalInterface
interface Transformer<T, R> {
  R transform(T t);
}
```

A key advantage of a functional interface is that there is no ambiguity about which method is being overridden by an implementing subclass.

For instance, consider:
```Java
Transformer<Integer, Integer> square = new Transformer<>() {
  @Override
  public Integer transform(Integer x) {
    return x * x;
  }
};
Transformer<Integer, Integer> incr = new Transformer<>() {
  @Override
  public Integer transform(Integer x) {
    return x + 1;
  }
};
```

You can see that there is much boilerplate code in the two functions above that we can remove.  Since we are assigning it to a variable of type `Transformer` interface, we don't have to write `new Transformer<>() { .. }`.  And since `Transformer` is an interface, there is no constructor.  Since there is only one abstract method to overwrite, we don't have to write `@Override public Integer transform(..) { .. }`.

What remain after we eliminate the obvious boilerplate code are (i) the parameter `Integer x` and (ii) the body of `transform`, which is `{ return x * x; }`.    We can use the Java arrow notation `->` to now link the parameter and the body:

```Java
Transformer<Integer, Integer> square = (Integer x) -> { return x * x; };
Transformer<Integer, Integer> incr = (Integer x) -> { return x + 1; };
```

You might notice that the type of the parameter is redundant as well since the type argument to `Transformer` already tells us this function takes in an `Integer`. We can further simplify it to:
```Java
Transformer<Integer, Integer> square = (x) -> { return x * x; };
Transformer<Integer, Integer> incr = (x) -> { return x + 1; };
```

or simply:
```Java
Transformer<Integer, Integer> square = x -> { return x * x; };
Transformer<Integer, Integer> incr = x -> { return x + 1; };
```

where there is only one parameter.

Since the body has only a single return statement, we can simplify it further:
```Java
Transformer<Integer, Integer> square = x -> x * x;
Transformer<Integer, Integer> incr = x -> x + 1;
```

Now, that's much better!

The expressions above, including `x -> x * x`, are called _lambda expressions_.  You can recognize one by the use of `->`.   The left-hand side lists the parameters (use `()` if there is no parameter), while the right-hand side is the computation.  We do not need the type in cases where Java can infer the type, nor need the return keyword and the curly brackets when there is only a single return statement.

!!! note "lambda"
    Alonzo Church invented lambda calculus ($\lambda$-calculus) in 1936, before electronic computers, as a way to express computation.  In $\lambda$-calculus, all functions are anonymous.  The term lambda expression originated from there.

### Method Reference

Lambda expression is useful for specifying a new anonymous method.  Sometimes, we want to use an existing method as a first-class citizen instead.

Recall the `distanceTo` method in `Point`, which takes in another point as a parameter and returns the distance between this point and the given point.

```Java
class Point {
    :
    
  public double distanceTo(Point p) {
      :
  }
}
```

We can write our `Transformer` like this using an anonymous class:
```Java
Point origin = new Point(0, 0);
Transformer<Point, Double> dist = new Transformer<>() {
  @Override
  public Double transform(Point p) {
    return origin.distanceTo(p);
  }
}
```

or using a lambda expression:
```Java
Point origin = new Point(0, 0);
Transformer<Point, Double> dist = p -> origin.distanceTo(p);
```

but since `distanceTo` takes in one parameter and returns a value, it already fits as a transformer, and we can write it as:
```Java
Point origin = new Point(0, 0);
Transformer<Point, Double> dist = origin::distanceTo;
```

The double-colon notation `::` is used to specify a _method reference_.  We can use method references to refer to a (i) static method in a class, (ii) instance method of a class or interface, (iii) constructor of a class.  Assume the following variable declaration

```java
A a = new A( .. );
```

Here are some examples of the double-colon notation and their equivalent lambda expression.

```Java
A::f    // x -> A.f(x)
A::new  // x -> new A(x)
a::g    // x -> a.g(x)
A::h    // (x, y) -> x.h(y) or (x, y) -> A.h(x, y)
```

The last example shows that the same method reference expression can be interpreted in two different ways.  The actual interpretation depends on how many parameters `h` takes and whether `h` is a class method or an instance method.  When compiling, Java searches for the matching method, performing type inferences to find the method that matches the given method reference.  A compilation error will be thrown if there are multiple matches or if there is ambiguity in which method matches.

!!! danger "Not Effectively Final"
    Consider the two examples above again, reproduced below.

    === "Lambda"
        ```java
        Point origin = new Point(0, 0);
        Transformer<Point, Double> dist = p -> origin.distanceTo(p);
        ```
    === "Method Reference"
        ```java
        Point origin = new Point(0, 0);
        Transformer<Point, Double> dist = origin::distanceTo;
        ```
    
    In the case of lambda, the variable `origin` is _effectively final_.  On the other hand, the variable `origin` is not effectively final when using method reference.  In fact, the following is allowed.

    ```java
    Point origin = new Point(0, 0);
    Transformer<Point, Double> dist = origin::distanceTo;
    origin = new Point(1, 1);
    double diagonal = dist.transform(new Point(3, 4)); // 5.0
    ```

    Note that Line 4 is computing the distance with respect to the original `origin` at (0, 0).  This is the famous Pythagorean triple 3, 4, 5.  The distance between the point (0, 0) and the point (3, 4) is exactly 5.  So although `origin` is not effectively final, the method reference `origin::distanceTo` is pointing to exactly the method `distanceTo` within this particular instance.  So even if the variable `origin` is reassigned to another value, the method reference is still pointing to the old method.

### Lexical This

In our discussion above -- as well as our mental model -- we think of lambda as a _syntactic sugar_ for anonymous class.  Which makes the explanation neat because we implement anonymous function (_i.e., lambda_) using something else that is anonymous.

Unfortunately, the reality is not so clean and tidy because Java compiler performs quite a number of optimizations that may cause such mental model to breakdown.  The following is an example of when such mental model breakdown for a good reason.

Consider the following code:

```java
class A {
  private int x;

  public A(int x) {
    this.x = x;
  }

  public int getX() {
    Transformer<Integer, Integer> f = y -> this.x;
    return f.transform(0);
  }
}
```

It is _reasonable_ to assume that the following code compiles.

```java
A a = new A(2);
a.getX(); // 2
```

And indeed, the code above compiles and the result of `a.getX()` is indeed 2.  However, this is a violation of our mental model where lambda is converted into an anonymomus class.  The rewriting of the code above using anonymous class will be the following:

```java
class A {
  private int x;

  public A(int x) {
    this.x = x;
  }

  public int getX() {
    Transformer<Integer, Integer> f = new Transformer<>() {
      @Override
      public Integer transform(Integer y) {
        return this.x;
      }
    };
    return f.transform(0);
  }
}
```

Unfortunately this code does not compile.  In fact, we will get the following error

```
_.java:_: error: cannot find symbol
        return this.x;
                   ^
  symbol: variable x
1 error
```

The problem here is that the keyword `this` is referring to two different instances.  In the case of lambda, it is referring to the instance of class `A`.  On the other hand, in the case of anonymous class, `this` is referring to the instance of the anonymous class.  So it causes the error because the anonymous class has no field called `x`.

However, in a way, it is reasonable when using lambda not to think about the presence of the anonymous class.  This is what we meant by lexical `this`.  So now we are faced with a slight conundrum because the mental model of having lambda as a syntactic sugar of anonymous class is _useful_ but unfortunately _wrong_.

Due to its usefulness, we will keep using the mental model of lambda being a syntactic sugar for anonymous class when tracing using stack and heap diagram.  However, the keyword `this` should be referring to the correct instance as per lexical `this`.

## Curried Functions

Mathematically, a function takes in only one value and returns one value (_e.g.,_ `square` _above_).  In programming, we often need to write functions that take in more than one argument (_e.g.,_ `add` _above_).
Even though `Transformer` only supports function with a single parameter, we can build functions that take in multiple arguments.  Let's look at this mathematically first.  Consider a binary function $f: (X, Y) \rightarrow Z$.  We can introduce $F$ as a set of all functions $f': Y \rightarrow Z$, and rewrite $f$ as $f: X \rightarrow F$, or $f: X \rightarrow Y \rightarrow Z$.

The arrow $\rightarrow$ is to be read from right-to-left.  So $f: X \rightarrow Y \rightarrow Z$ is equivalent to $f: X \rightarrow (Y \rightarrow Z)$.  But what does it actually mean?  It means that instead of having a function that takes in two arguments, we can instead have a function that takes in one argument (_typically the first argument_) and return another function to accept the second argument.

A trivial example of this is the `add` method that adds two `int` values.
```Java
int add(int x, int y) {
  return x + y;
}
```

This can be written as
```Java
Transformer<Integer, Transformer<Integer, Integer>> add = x -> y -> (x + y);
```

The way we read `x -> y -> (x + y)` is actually from _right-to-left_.  So, it is equivalent to `x -> (y -> (x + y))`.  We can then reason about this from innermost to outermost:

- __Innermost:__ `y -> (x + y)` is a function that takes in a single argument `y` and returns the sum of `(x + y)`.  Let us call this function `f_inner`.
- __Outermost:__ `x -> f_inner` is a function that takes in a single argument `x` and returns the innermost function `f_inner`.

Combining the two, we can read it as follows:

> `x -> y -> (x + y)` is a function that takes in a single argument `x` and returns another function.  The returned function takes in a single argument `y` and returns the result of adding `x` (_the argument accepted by the outer function_) with `y` (_the argument accepted by the inner function_).

To calculate 1 + 1, we call
```Java
add.transform(1).transform(1);
```

Let's break it down again, `add` is a function that takes in an `Integer` object and returns a unary `Function` over `Integer`.  So `add.transform(1)` returns the function `y -> 1 + y`.  We could assign this to a variable:
```Java
Transformer<Integer,Integer> incr = add.transform(1);
```

Note that `add` is no longer a function that takes two arguments and returns a value.  It is a _higher-order function_ that takes in a single argument and returns another function.

The technique that translates a general $n$-ary function to a sequence of $n$ unary functions is called _currying_.  After currying, we have a sequence of _curried_ functions.  

!!! note "Curry"
    Currying is not related to food but rather is named after computer scientist Haskell Curry, who popularized the technique.

How is currying useful?  Consider `add(1, 1)` -- we have to have both arguments available at the same time to compute the function.  With currying, we no longer have to.  We can evaluate the different arguments at a different time (_as_ `incr` _example above_).  This feature is useful in cases where some arguments are not available until later.  We can _partially apply_ a function first.  This is also useful if one of the arguments does not change often, or is expensive to compute.  We can save the partial results as a function and continue applying later.  We can dynamically create functions as needed, save them, and invoke them later.

!!! info "Instance Method as Partially Applied Class Method"
    One popular point of view is that an instance method is a partially applied static method with an additional _first_ parameter called `this`.  Take for instance an instance method `C::f(A1, A2)` in class `C` that takes in two parameters of type `A1` and `A2`.  This can also be viewed as a class method (_i.e., with_ `static` _keyword_) of the form `C::f(C, A1, A2)`.  So instead of taking in two parameters, it takes in three parameters.  We assume that the first parameter `C` is called `this`.

    Of course, this view is just an approximation but it is a useful one.

## Lambda as Closure

In the example, we showed earlier,

```Java
Point origin = new Point(0, 0);
Transformer<Point, Double> dist = p -> origin.distanceTo(p);
```

the variable `origin` is captured by the lambda expression `dist`.  Just like in local and anonymous classes, a captured variable must be either explicitly declared as `final` or is effectively final.

A lambda expression stores more than just the function to invoke -- it also stores the data from the environment where it is defined.  We call such a construct that stores a function together with the enclosing environment a _closure_.

Being able to save the current execution environment, and then continue to compute it later, adds new power to how we can write our program.  We can make our code cleaner with fewer parameters to pass around and less duplicated code.  We can separate the logic to do different tasks in a different part of our program easier.  Unfortunately, as the captured variables are effectively final, we cannot modify them so the power we gain is more restricted.

We will see more examples of this later.
