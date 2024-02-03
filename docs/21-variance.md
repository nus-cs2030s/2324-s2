# Unit 21: Variance

!!! abstract "Learning Objectives"

    After taking this unit, students should:

    - understand the definition of the variance of types: covariant, contravariant, and invariant
    - be aware that the Java array is covariant and how it could lead to run-time errors that cannot be caught during compile time

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

Line 4 is not surprising since the type for `objArray` matches that of parameter `array`.  Line 5, however, shows that it is possible to assign an instance with run-time type `Integer[]` to a variable with compile-time type `Object[]`.

## Variance of Types

So far, we have established the subtype relationship between classes and interfaces based on inheritance and implementation.  The subtype relationship between _complex types_ such as arrays, however, is not so trivial.  Let's look at some definitions.

The _variance of types_ refers to how the subtype relationship between complex types relates to the subtype relationship between components.

Let $C(S)$ corresponds to some complex type based on type $S$.  An array of type $S$ is an example of a complex type.

We say a complex type is:

- _covariant_ if $S <: T$ implies $C(S) <: C(T)$
- _contravariant_ if $S <: T$ implies $C(T) <: C(S)$
- _invariant_ if it is neither covariant nor contravariant.

## Java Array is Covariant

 Arrays of reference types are covariant in Java[^1].  This means that, if $S <: T$, then $S[] <: T[]$.  

[^1]: Arrays of primitive types are invariant.

For example, because `Integer` <: `Object`, we have `Integer[]` <: `Object[]` and we can do the following:

```Java
Integer[] intArray;
Object[] objArray;
objArray = intArray; // ok
```

By making array covariant, however, Java opens up the possibility of run-time errors, even without typecasting!

Consider the following code:
```Java
Integer[] intArray = new Integer[2] {
  Integer.valueOf(10), Integer.valueOf(20)
};
Object[] objArray;
objArray = intArray;
objArray[0] = "Hello!"; // <- compiles!
```

On Line 5 above, we set `objArray` (with a compile-time type of `Object[]`) to refer to an object with a run-time type of `Integer[]`.  This is allowed since the array is covariant.

On Line 6, we try to put a `String` object into the `Object` array.  Since `String` <: `Object`, the compiler allows this.  The compiler does not realize that at run-time, the `Object` array will refer to an array of `Integer`.  

So we now have a perfectly compilable code, that will crash on us when it executes Line 6 -- only then would Java realize that we are trying to stuff a string into an array of integers!

This is an example of a type system rule that is unsafe.  Since the array type is an essential part of the Java language, this rule cannot be changed without ruining existing code.  We will see later how Java avoids this pitfall for other complex types (such as a list).
