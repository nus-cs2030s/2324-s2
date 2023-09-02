# Unit 21: Variance

!!! abstract "Learning Objectives"

    Students should

    - understand the definition of the variance of types: covariant, contravariant, and invariant.
    - be aware that the Java array is covariant and how it could lead to run-time errors that cannot be caught during compile time.

## Motivation

Both the methods `findLargest` and `contains` takes in an array of reference types as parameters:

```Java
// version 0.4
GetAreable findLargest(GetAreable[] array) {
  double maxArea = 0;
  GetAreable maxObj = null;
  for (GetAreable curr : array) {
    double area = curr.getArea();
    if (area > maxArea) {
      maxArea = area;
	  maxObj = curr;
    }
  }
  return maxObj;
}

// version 0.1 (with polymorphism)
boolean contains(Object[] array, Object obj) {
  for (Object curr : array) {
    if (curr.equals(obj)) {
      return true;
    }
  }
  return false;
}
```

What are some possible arrays that we can pass into these methods?  Let's try this:
```Java
Object[] objArray = new Object[] { Integer.valueOf(1), Integer.valueOf(2) };
Integer[] intArray = new Integer[] { Integer.valueOf(1), Integer.valueOf(2) };

contains(objArray, Integer.valueOf(1)); // ok
contains(intArray, Integer.valueOf(1)); // ok
```

Line 4 is not surprising since the type for `objArray` matches that of parameter `array`.  Line 5, however, shows that it is possible to assign an instance with run-time type `Integer[]` to a variable with compile-time type `Object[]`.  [We have explained the reason for this before](14-polymorphism.md) and that is because for every reference type that is declared, Java automatically create an array type for that reference type such that follows the subtyping relationship of the reference type.  In this case, since `Integer` <: `Number` <: `Object`, Java automatically create the corresponding array types `[LInteger;` <: `[LNumber;` <: `[LObject;`.

For simplicity, from this point onwards, if we have a reference type `T`, we will be using `T[]` to indicate the automatically generated array type for `T`.  Now, note that we only create the array type for reference type.  This means that for primitive type, the developer of Java language have to add special types corresponding to the array of primitive type.

## Variance of Types

So far, we have established the subtype relationship between classes and interfaces based on inheritance and implementation.  The subtype relationship between _complex types_ such as arrays, however, is not so trivial.  Let's look at some definitions.

The _variance of types_ refers to how the subtype relationship between complex types relates to the subtype relationship between components.

Let $C(S)$ corresponds to some complex type based on type $S$.  An array of type $S$ is an example of a complex type.

We say a complex type is:

- _covariant_ if $S <: T$ implies $C(S) <: C(T)$
- _contravariant_ if $S <: T$ implies $C(T) <: C(S)$
- _invariant_ if it is neither covariant nor contravariant.

!!! tips "How to Memorize"
    Think of _variant_ as an arrow (_i.e., it has direction_).  Then, the word "_contra_" should evoke the feeling of "counter".  In other words, the direction should be in the counter (_i.e., opposite_) direction.  That's why for _contravariant_, we have $S <: T$ implies $C(S) :> C(T)$.  Of course, it is actually written as $C(T) <: C(S)$.

    The word "_co_" should evoke the same direction (_like co-payment, co-operation, etc_).  So, the direction should be in the same direction.  That's why for _covariant_, we have $S <: T$ implies $C(S) <: C(T)$.

    Lastly, invariant is simply neither.

## Java Array is Covariant

 Arrays of reference types are covariant in Java[^1].  This means that, if $S <: T$, then $S[] <: T[]$.  

[^1]: Arrays of primitive types are invariant.

For example, because `Integer` <: `Object`, we have `Integer[]` <: `Object[]` and we can do the following:

=== "Code"
    ```Java
    Integer[] intArray;
    Object[] objArray;
    objArray = intArray; // ok
    ```

=== "What Compiler Sees"
    ```Java
    Integer[] intArray;  // intArray::Integer[]
    Object[] objArray;   // objArray::Object[]
    objArray = intArray; // objArray::Object[]  <- intArray::Integer[] (ok because Integer[] <: Object[])
    ```

By making array covariant, however, Java opens up the possibility of run-time errors, even without typecasting!

Consider the following code:

=== "Code"
    ```Java
    Integer[] intArray = new Integer[2] {
      Integer.valueOf(10), Integer.valueOf(20)
    };
    Object[] objArray;
    objArray = intArray;
    objArray[0] = "Hello!"; // <- compiles!
    ```

=== "What Compiler Sees"
    ```Java
    Integer[] intArray = new Integer[2] {
      Integer.valueOf(10), Integer.valueOf(20)
    };                      // intArray::Integer[]
    Object[] objArray;      // objArray::Object[]
    objArray = intArray;    // objArray::Object[]  <- intArray::Integer[]
    objArray[0] = "Hello!"; // objArray[0]::Object <- "Hello!"::String (ok, because String <: Object)
    ```

On Line 5 above, we set `objArray` (_with a compile-time type of_ `Object[]`) to refer to an object with a run-time type of `Integer[]`.  This is allowed since the array is covariant.

On Line 6, we try to put a `String` object into the `Object` array.  Since `String` <: `Object`, the compiler allows this.  The compiler does not realize that at run-time, the `Object` array will refer to an array of `Integer`.  

So we now have a perfectly compilable code, that will crash on us when it executes Line 6 -- only then would Java realize that we are trying to stuff a string into an array of integers!

This is an example of a type system rule that is unsafe.  Since the array type is an essential part of the Java language, this rule cannot be changed without ruining existing code.  We will see later how Java avoids this pitfall for other complex types (_such as a list_).

## Producer/Consumer

Variance is closely related to the concept of producer/consumer.  We will encounter this concept again later.  For now, we can simply introduce the following _obvious_ definition.

!!! info "Producer"
    Producer _produces_ value.

!!! info "Consumer"
    Consumer _consumes_ value.

We will show two different of producer/consumer.

| Kind | as Producer | as Consumer |
|------|-------------|-------------|
| _Array_ | `X x = arr[n];` | `arr[n] = value;` |
| _Function_ | `X x = f(arg);` (_i.e., return value is produced_) | `f(value);` (_i.e., argument is consumed_) |

## Type Problems

There are some type problems related to producer/consumer and variance of types.  We have seen one such example above.  Due to covariance of Java array, there are some run-time errors that cannot be detected by our compiler.  Notice that this happens when we treat the array as a _consumer_.  We can actually state this more generally,

> There will be some run-time errors that cannot be detected by compiler when having producer covariant.

A more general example is as follows but we will illustrate using array.  Consider `A1` <: `B` and `A2` <: `B`.  By covariance, `A[]`
 <: `B[]`.  Then the following code will not produce compilation error but will produce run-time error.

```java
A1[] aArr = new A1[] { new A1(), new A1() };
B[] bArr = aArr;    // assume covariant: A1[] <: B[]
bArr[0] = new A2(); // compiles because A2 <: B
                    // but this is a run-time error
```

We can also state the opposite problem.

> There will be some run-time errors that cannot be detected by compiler when having consumer contravariant.

We will use the same subtyping relationship as above.  However, note that __the following code is simply hypotheticals__ because Java array is not contravariant.

```java
B[] bArr = new B[] { new A1(), new A2() };
A1[] aArr = bArr;   // assume contravariant: B[] <: A1[]
A1 a1 = aArr[2];    // compiles because A1 <: A1
                    // but this is a run-time error
```