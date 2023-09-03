# Unit 20: Run-Time Class Mismatch

!!! abstract "Learning Objectives"

    Students should

    - understand the need for narrowing type conversion and type casting when writing code that depends on higher-level abstraction.
    - understand the possibility of encountering run-time errors if typecasting is not done properly.

## Problem

We have seen in [Unit 18](18-interface.md) how we can write code that is reusable and general by making our code dependent on types at a higher-level of abstraction.  Our main example is the following `findLargest` method, which takes in an array of objects that support the `getArea` method, and returns the largest area among these objects.

```Java
// version 0.3
double findLargest(GetAreable[] array) {
  double maxArea = 0;
  for (GetAreable curr : array) {
    double area = curr.getArea();
    if (area > maxArea) {
      maxArea = area;
    }
  }
  return maxArea;
}
```

The method served our purpose well, but it is NOT a very well-designed method.  Just returning the value of the largest area is not as useful as returning the _object_ with the largest area.  Once the caller has a reference of the object, the caller can call `getArea` to find the value of the largest area.

Let's write our `findLargest` method to find which object has the largest area instead.  

```Java hl_lines="2 4 9 12"
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
```

Let's see how `findLargest` can be used:

```Java
GetAreable[] circles = new GetAreable[] {
  new Circle(new Point(1, 1), 2),
  new Circle(new Point(0, 0), 5)
};

GetAreable ga = findLargest(circles);  // ok
Circle c1 = findLargest(circles); // error
Circle c2 = (Circle) findLargest(circles); // ok
```

The return type of `findLargest` (_version 0.4_) is now `GetAreable`.  On Line 6 above, we assign the return object with a compile-time type of `GetAreable` to `ga`, which also has `GetAreable` as its compile-time type.  Since the variable `ga` is of type `GetAreable`, however, it is not very useful.  Recall that `GetAreable` is an interface with only one method `getArea`.  We cannot use it as a circle.

On Line 7, we try to return the return object to a variable with compile-time type `Circle`.  This line, however, causes a compile-time error.  Since `Circle` <: `GetAreable`, this is a narrowing type conversion and thus is not allowed (_See [Unit 14](14-polymorphism.md)_).  We will have to make an explicit cast of the result to `Circle` (_on Line 8_).  Only with casting, our code can compile and we get a reference with a compile-time type of `Circle`.

## Cast Carefully

Typecasting, as we did in Line 8 above, is basically is a way for programmers to ask the compiler to trust that the object returned by `findLargest` has a run-time type of `Circle` (_or its subtype_).

In the snippet above, we can be sure (_even **prove**_) that the returned object from `findLargest` must have a run-time type of `Circle` since the input variable `circles` contains only `Circle` objects.

The need to cast our returned object, however, leads to fragile code.  Since the correctness of Line 8 depends on the run-time type, the compiler cannot help us.  It is then up to the programmers to not make mistakes.  This is similar to the reasoning we used in [Unit 11](11-inheritance.md#compile-time-type) for why we only use the compile-time type information for type checking.  We cannot guarantee that this will work in general if we change the initialization of the `GetAreable` array.

Consider the following two snippets, which will compile perfectly, but will lead to the program crashing at run-time.

```Java
GetAreable[] circles = new GetAreable[] {
  new Circle(new Point(1, 1), 2),
  new Square(new Point(1, 1), 5)
};

Circle c2 = (Circle) findLargest(circles);
```

Or

```Java
GetAreable[] circles = new GetAreable[] {
  new Circle(new Point(1, 1), 2),
  new Circle(new Point(1, 1), 5)
};

Square sq = (Square) findLargest(circles);
```

We will see how to resolve this problem in later units.

!!! info "Actual Casting"

    Note that at run-time, there is no need to do an actual casting from one type to another.  Remember that at run-time we will be using the run-time type information.  This run-time type information does not change!

    What type casting does is to produce an expression such that the compile-time type of the value produced by the expression is now of the specific type as specified in the type cast.  Consider the following code snippet.

    ```java
    String str1 = "CS2030S";
    Object obj = s; // String <: Object
    String str2 = (String) obj;
    ```

    Here, the expression `(String) obj` produces a value with compile-time type of `String`.  This value is then assigned to `str2`.  This is allowed because `String` (_from_ `(String) obj`) <: `String` (_from the compile-time type of_ `str2`).  Note that the compile-time type of both `str1` and `obj` are __unchanged__.  They are still `String` and `Object` respectively.

    These compile-time type information is needed only for compilation and is no longer needed at run-time.  So at run-time, what Java is actually doing is to check if the given run-time type is actually a subtype of the declared casted type.  This is done without changing the run-time type of the object.  However, if the check fails, then you will get a run-time error `ClassCastException`.