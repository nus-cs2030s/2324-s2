# Unit 28: Immutability

!!! abstract "Learning Objectives"

    Students should

    - understand when a class is immutable or not.
    - be able to create an immutable class.

## Avoiding Change

So far in this course, we have been focusing on three ways of dealing with software complexity: (i) by encapsulating and hiding the complexity behind abstraction barriers, (ii) by using a language with a strong type system and adhering to the subtyping substitution principle, and (iii) applying the abstraction principles and reusing code written as functions, classes, and generics types.

Another useful strategy to reduce bugs when code complexity increases is to avoid change altogether.  This can be done by making our classes _immutable_.  We create an instance of an immutable class, the instance _cannot have any visible changes outside its abstraction barrier_.  This means that every call of the instance's method must behave the same way throughout the lifetime of the instance.

!!! info "Immutable Class"
    An immutable class is a class for which there cannot be any visible changes outside of its abstraction barrier.

There are many advantages of why we want to make our class immutable when possible.  To start, let's revisit a common bug due to aliasing.  Recall the following example from [Unit 9](09-composition.md),  where we create two circles `c1` and `c2` centered at the origin (0, 0).

```Java
Point p = new Point(0, 0);
Circle c1 = new Circle(p, 1);
Circle c2 = new Circle(p, 4);
```

Let's say that we have the `moveTo` method in both `Circle` and `Point`, to move the circle and point respectively.

```Java
class Point {
  private double x;
  private double y;

    :

  public void moveTo(double x, double y) {
    this.x = x;
    this.y = y;
  }
}

class Circle {
  private Point c;
  private double r;

  public Circle (Point c, double r) {
    this.c = c;
    this.r = r;
  }

    :

  public void moveTo(double x, double y) {
    c.moveTo(x, y);
  }
}
```

Suppose we want to move `c1` __and only__ `c1` to be centered at (1,1).

```Java
c1.moveTo(1, 1);
```

The line of code above surprisingly moved the center of _both_ `c1` and `c2`, due to both circles `c1` and `c2` sharing the same point.  We have explored a solution below:

```Java
Point p1 = new Point(0, 0);
Circle c1 = new Circle(p1, 1);

Point p2 = new Point(0, 0);
Circle c2 = new Circle(p2, 4);

c1.moveTo(1, 1);
```

This approach avoids sharing references by creating copies of our points so that no two references point to the same instance, avoiding aliasing altogether.  This _partial_ fix, however, comes with extra costs in computational resources as the number of objects may proliferate.

This is also not a complete solution because surprisingly, we can move `c2` without calling `c2.moveTo(1, 1)` but by calling the code below.

```Java
p2.moveTo(1, 1);
```

Let's now see how immutability can help us resolve our problem.


## Immutable Points and Circles

### Immutable Points

Let's start by making our `Point` class immutable.  We start by making the fields `final` to signal our_ intention_ that we do not intend to _assign_ another value to them.  Now that the `x` and `y` cannot be re-assigned (_a new value or even the same value_), to move a point, we shouldn't re-assign to the fields `x` and `y` anymore.  Instead, we return a new `Point` instance to prevent mutating the current instance, as follows:

```Java
final class Point {
  private final double x;
  private final double y;

  public Point(double x, double y) {
    this.x = x;
    this.y = y;
  }

  public Point moveTo(double x, double y) {
    return new Point(x, y);
  }
    :

  @Override
  public String toString() {
    return "(" + this.x + "," + this.y + ")";
  }
}
```

Note that, to avoid (_likely malicious or ignorant_) subclasses of `Point` overriding the methods to make it appears that the point has mutated, it is necessary that we declare immutable classes as `final` to disallow inheritance.

#### Analysis of Points

It is always good to check if the `Point` class above are really immutable.  Assume that the public methods available are:

- `Point Point::moveTo(double, double)`
- `String Point::toString()`
- `double Point::getX()`
- `double Point::getY()`

To see why `Point` is indeed is immutable, we look at the following code snippet.

```java
Point p = new Point(1.0, 1.0);
p.getX(); // 1.0
  :       // any sequence of invocation of methods in Point`
p.getX(); // 1.0
```

No methods in `Point` can actually change the field `x` or `y`.  First, notice that `x` and `y` are `double`.  So the only way they can be changed is by assignment to the fields.  But the fields are declared `final`, so there can be no assignment done on them.

The keyword `final` on the fields are not _necessary_ because there is no assignment at all to the fields.  However, it is still a good practice to have them.  Just in case someone in the future try to perform an assignment on the field and change the `Point` class from immutable to mutable.

### Immutable Circle

Now, let's make `Circle` immutable:

```Java
final class Circle {
  private final Point c;
  private final double r;

  public Circle (Point c, double r) {
    this.c = c;
    this.r = r;
  }
    :

  public Circle moveTo(double x, double y) {
    return new Circle(c.moveTo(x, y), this.r);
  }
}
```

#### Analysis of Circle

With `Point` being immutable, we can be sure that the only way for the field `c` to be changed is by assigning a new value to `c`.  However, this is again prevented by the keyword `final` in the field `c`.  And again, even without such keyword, there is still no assignment on it.  So we can be sure that `Circle` is also immutable.

In other words, once an instance is created, it remains unchanged (_outside the abstraction barrier_).

```Java
Point p = new Point(0, 0);
Circle c1 = new Circle(p, 1);
Circle c2 = new Circle(p, 4);
c1.moveTo(1, 1); // c1 remains unchanged
```

To update the variable `c1`, we need to explicitly reassign it.

```Java
c1 = c1.moveTo(1, 1);
```

Now, `c1` moves to a new location, but `c2` remains unchanged.

### Comparison with Mutable Version

Compare our new immutable approach to the two approaches above. The first shares all the references and is bug-prone.  The second creates a new copy of the instance every time and is resource-intensive.  Our third approach, using immutable classes, allows us to share all the references until we need to modify the instance, in which case we make a copy.  Such a _copy-on-write_ semantic allows us to avoid aliasing bugs without creating excessive copies of objects.

Note that the `final` keyword prevents assigning new value to the field.  Unfortunately, it does not prevent the field from being mutated.  So, to ensure that the classes we create are immutable, we have to ensure that the fields are themselves immutable.

## Necessity of Final Keyword on Class

We mentioned that it is necessary that we declare immutable classes as `final` to disallow inheritance.  So let us elaborate this further.  Consider the immutable `Point` from above.  Now let us create a subclass of it.

```java
class MutablePoint extends Point {
  private double x;
  private double y;

  public MutablePoint(double x, double y) {
    super(x, y);
    this.x = x;
    this.y = y;
  }

  @Override
  public Point moveTo(double x, double y) {
    this.x = x;
    this.y = y;
    return super.moveTo(x, y);
  }
    :

  @Override
  public String toString() {
    return "(" + this.x + "," + this.y + ")";
  }
}
```

You can see that we kept a duplicate of the internals of `Point` in `MutablePoint`, which is a bad practice.  But it also shows that if we allow inheritance, we may end up with the following behavior that shows the mutability of `Point`.

```java
Point p = new MutablePoint(1, 1); // at (1, 1)
p.moveTo(2, 2); // now it is at (2, 2)
```

Although `Point` is immutable, its subclass `MutablePoint` is actually mutable.  This also breaks LSP because the subclass breaks the expectation that the superclass is immutable.

## Advantages of Being Immutable

We have seen how making our classes immutable helps us remove the risk of potential bugs when we use composition and aliasing.  Immutability has other advantages as well.  

### Ease of Understanding

Code written with immutable objects is easier to reason with and easier to understand.  Suppose we create a `Circle` and assign it to a local variable:

```Java
Circle c = new Circle(new Point(0, 0), 8);
```

We pass `c` around to many other methods.  These other methods may invoke `c`'s methods; we may invoke `c`'s methods locally as well.  But, despite putting `c` through so much, unless we have explicitly re-assigned `c`, we can guarantee that `c` is still a circle centered at (0,0) with a radius of 8.  This immutable property makes it significantly easier to read, understand, and debug our code.

Without this property, we have to trace through all the methods that we pass `c` to, and each call of `c`'s methods to make sure that none of these codes modifies `c`.

### Enabling Safe Sharing of Objects

Making a class immutable allows us to safely share instances of the class and therefore reducing the need to create multiple copies of the same object.  For instance, the origin (0, 0) is commonly used.  If the instance is immutable, we can just create and cache a single copy of the origin, and always return this copy when the origin is required.

Let modify our `Point` class so that it creates a single copy of the origin and returns the same copy every time the origin is required.

```Java
final class Point {
  private final double x;
  private final double y;
  
  private Point(double x, double y) {
    this.x = x;
    this.y = y;
  }

  private final static Point ORIGIN = new Point(0, 0);

  public static Point of(double x, double y) {
    if (x == 0 && y == 0) {
      return ORIGIN;
    }
    return new Point(x, y);
  }
    :
}
```

We made a few changes in the above:

- We made the constructor for `Point` private so that one cannot call the constructor directly.
- We provide a class factory method named `of` for the client to create a `Point` instance.  The `of` method returns the same instance `ORIGIN` every time `Point.of(0, 0)` is called.

Such a design pattern is only safe when the class is immutable.  Consider the mutable version of `Point` -- calling `Point.of(0, 0).moveTo(1, 1)` would change every reference to the origin to (1, 1), causing chaos in the code!

### Enabling Safe Sharing of Internals

Immutable instances can also share their internals freely.  Consider an immutable implementation of our `Array<T>`, called `ImmutableArray<T>`.  Let's start with a simple version first.

```Java
// version 0.1
final class ImmutableArray<T> {
  private final T[] array;

  // Only items of type T goes into the array.
  @SafeVarargs
  public static <T> ImmutableArray<T> of(T... items) {
    // We need to copy to ensure that it is truly immutable
    // Explanation will come later
    @SuppressWarnings("unchecked");
    T[] arr = (T[]) new Object[items.length];
    for (int i=0; i<items.length; i++) {
      arr[i] = items[i];
    }
    return new ImmutableArray<>(arr);
  }

  private ImmutableArray(T[] a) {
    this.array = a;
  }

  public T get(int index) {
    return this.array[index];
  }
}
```

There are a few things to note here.

**Varargs.** &nbsp;&nbsp; The parameter to the class factory method `of` has the form `T... items`.  The triple `.` notation is a Java syntax for a variable number of arguments of the same type (`T`).  Often called _varargs_, this is just syntactic sugar for passing in an array of items to a method.  The method is called _variadic method_.  We can then call `of` with a variable number of arguments, such as:

```Java
ImmutableArray<Integer> a;
a = ImmutableArray.of();
a = ImmutableArray.of(1, 2, 3);
a = ImmutableArray.of(1, 2, 3, 4, 5);
```

We can also call `of` with a single array as an argument!

```java
ImmutableArray<Integer> a;
a = ImmutableArray.of(new Integer[] { 1, 2, 3 });
```

!!! info "Another Main Function"
    Now that we have learnt about _vargargs_, we can write a different version of the main function.  In this version, instead of having a parameter of an array of `String` written as `String[]`, we have a parameter of an array of `String` written as `String...`.

    ```java
    public static void main(String... args) {
        :
    }
    ```

**@SafeVarargs.** &nbsp;&nbsp; Since the varargs is just an array, and array and generics do not mix well in Java, the compiler would throw us an unchecked warning.  In this instance, however, we know that our code is safe because we never put anything other than items of type `T` into the array.  We can use the `@SafeVarargs` annotation to tell the compiler that we know what we are doing and this varargs is safe.

Notice that we removed the `set` method and there is no other way an external client can modify the array once it is created.  This, of course, assumes that we will only be inserting an immutable object into our immutable array.  Unfortunately, this cannot be enforced by the compiler as the generic type `T` can be anything.

Now, suppose that we wish to support a `subarray` method, that returns a new array containing only a range of elements in the original array.  It behaves as follows:

```Java
ImmutableArray<Integer> a = ImmutableArray.of(10, 20, 30, 40, 50, 60);
ImmutableArray<Integer> b = a.subarray(2, 4); // b is [30, 40, 50]
b.get(0) // returns 30
ImmutableArray<Integer> c = b.subarray(1, 2); // c is [40, 50]
c.get(1) // returns 50
```

A typical way to implement `subarray` is to allocate a new `T[]` and copy the elements over.  This operation can be expensive if our `ImmutableArray` has millions of elements.  But, since our class is immutable and the internal field `array` is guaranteed not to mutate, we can safely let `b` and `c` refer to the same `array` from `a`, and only store the starting and ending index.

```Java
class ImmutableArray<T> {
  private final int start;
  private final int end;
  private final T[] array;

  @SafeVarargs
  public static <T> ImmutableArray<T> of(T... items) {
    // We need to copy to ensure that it is truly immutable
    // Explanation will come later
    @SuppressWarnings("unchecked");
    T[] arr = (T[]) new Object[items.length];
    for (int i=0; i<items.length; i++) {
      arr[i] = items[i];
    }
    return new ImmutableArray<>(arr, 0, items.length-1);
  }

  private ImmutableArray(T[] a, int start, int end) {
    this.start = start;
    this.end = end;
    this.array = a;
  }

  public T get(int index) {
    if (index < 0 || this.start + index > this.end) {
      throw new IllegalArgumentException("Index out of bound");
    }
    return this.array[this.start + index];
  }

  public ImmutableArray<T> subarray(int start, int end) {
     return new ImmutableArray<>(this.array, this.start + start, this.start + end);
  }
}
```

#### Why Copy the Array

Now we need to explain why we need to copy the parameter `items` in our factory method `of`.  Consider not copying the content of the array.  Since we can invoke the factory method by passing an array as argument, we can do the following.

```java
Integer[] int_arr = new Integer[]{ 1, 2, 3 };
ImmutableArray<Integer> arr = ImmutableArray.of(int_arr);
arr.get(0); // 1
int_arr[0] = 9;
arr.get(0); // will be 9 if array is not copied since there is an aliasing
```

### Enabling Safe Concurrent Execution

We will explore concurrent execution of code towards the end of the module, but making our classes immutable goes a long way in reducing bugs related to concurrent execution.  Without going into details here (you will learn the details later), concurrent programming allows multiple threads of code to run in an interleaved fashion, in an arbitrary interleaving order.   If we have complex code that is difficult to debug to begin with, imagine having code where we have to ensure its correctness regardless of how the execution interleaves!  Immutability helps us ensure that regardless of how the code interleaves, our objects remain unchanged.

## Final &ne; Immutable

When creating an immutable class, we need to be careful to distinguish between the keywords that helps us avoid accidentally making things easily mutable and the actual concept of immutable class.  For instance, it is _insufficient_ to simply declare all fields with `final` keywords.  Just because we cannot accidentally update the field, does not mean that the field is immutable.  Consider the same `Circle` above but with a getter for the center point and now imagine that the `Point` is mutable.

```java
final class Circle {
  private final Point c;
  private final double r;

  public Circle (Point c, double r) {
    this.c = c;
    this.r = r;
  }
    :
    
  public Point getCenter() {
    return this.c;
  }

  public Circle moveTo(double x, double y) {
    return new Circle(c.moveTo(x, y), r);
  }
}
```

We can then simply retrieve the center point and mutate it externally.

```java
Circle c = new Circle(new Point(0, 0), 1);
c.getCenter().moveTo(1, 1); // assume mutable Point
```

On the other hand, it is not even _necessary_ to use the `final` keyword to make an immutable class.  We simply have to have a class that prevents any and all kinds of sharing by copying all the parameters before assigning them into the fields and copying all return value.  Assume that all classes has a correctly implemented `clone()` method.  Then the following `Circle` is immutable even with getter and no `final` keyword on the fields.  We still need the `final` keyword on the class to disallow inheritance.

```java
final class Circle {
  private Point c;
  private double r;

  public Circle (Point c, double r) {
    this.c = c.clone();
    this.r = r; // primitive, no need cloning
  }
    :
    
  public Point getCenter() {
    return this.c.clone();
  }

  public Circle moveTo(double x, double y) {
    return new Circle(c.moveTo(x, y), r);
  }
}
```

That is not to say that the `final` keyword is not important.  It helps accidental re-assignment and in some cases that is sufficient especially if the fields are of primitive type.  Once we have created one immutable class, we can then create other larger immutable classes by only using immutable classes as fields.

### Trivially Immutable Class

We can even have trivially immutable classes.  Consider our old `Pair<S, T>` class.

```java
class Pair<S,T> {
  private S first;
  private T second;

  public Pair(S first, T second) {
    this.first = first;
    this.second = second;
  }

  public S getFirst() {
    return this.first;
  }

  public T getSecond() {
    return this.second;
  }
}
```

Assuming that `S` and `T` are _immutable_, the generic class `Pair<S, T>` is also immutable.  This is achieved by simply having no mutator.  Additionally, there is no `String toString()` method that exposes the internal state.

Another way to have a trivially immutable class with mutator is to have no accessor to the mutated state.  If we never expose the mutated internal state, then there is no way to have any visible changes outside of its abstraction barrier.  The following `Counter` is an example.

```java
class Counter {
  private int val;
  private int ctx;

  public Counter(int val) {
    this.val = val;
    this.ctx = 0;
  }

  public int get() {
    this.ctx += 1;
    return this.val;
  }

  @Override
  public String toString() {
    return "{" + this.val + "}";
  }
}
```

Note that the internal state `ctx` changed but there is no way for us to expose this externally.  So using only its abstraction barrier (_i.e., the methods_ `get()` _and_ `toString()`), we can never see any visible changes.

!!! note "Checklist for Immutability"
    The following is a general guide to help you create immutable class.  Some of the items in the checklist may not be necessary but they are good to have.

    1. Ensure that all fields have the `final` modifier (_not necessary but good to have_).
    2. Ensure that the types of all the fields are immutable classes.
    3. Ensure that arrays are copied before assigning to a field.
    4. Ensure that there is no mutator.
        - If there was a mutator and you are modifying the class to be immutable, then you need to return a new instance instead.
    5. Ensure that the class has the `final` modifier to prevent inheritance.