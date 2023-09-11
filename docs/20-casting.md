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

## Type Case Checks

Although type casting is like telling the compiler that we -- the programmer -- know better, some cases are really indefensible that the compiler will know immediately that it is wrong.  The checks done during type casting in Java can be classified into two parts: compile-time check and run-time check.

We consider the following statement:

```java
a = (C) b;
```

### Compile-Time Check

During compile-time, the compile will perform the following checks:

1. Find the compile-time type of variable `b` (_denoted_ CTT(`b`)).
2. Check if there is a "_possibility_" that the run-time type of `b` (_denoted_ RTT(`b`)) is a subtype of `C` (_i.e.,_ RTT(`b`) <: `C`).  We will explain the possibilities more later.
    - If it is _impossible_, then exit with __compilation error__.
    - Otherwise, continue to step 3.
3. Find the compile-time type of variable `a` (_denoted_ CTT(`a`)).
4. Check if `C` is a subtype of CTT(`a`) (_i.e.,_ `C` <: CTT(`a`)).
    - If it is not, then exit with __compilation error__.
    - Otherwise, add __run-time check__ for RTT(`b`) <: `C`.

Note that step (1) and (2) is checking if the type cast operation (_i.e.,_ `(C) b`) can potentially happen or not.  Step (3) and (4) checks if the assignment (_i.e.,_ `a = <expr>;`) satisfies the subtyping relationship or not.  The check at step (4) is simply a check for _widening_.

!!! success "Possibility"
    We will consider 3 cases where it is _possible_ for RTT(`b`) to be a subtype of `C`.  There may be other cases, so you have to think about possibilities in terms of potential new classes added in the future.

    1. Case 1: CTT(`b`) <: `C`
        - This is simply widening and is always allowed.
        - The use of explicit type cast is unnecessary but not incorrect.
    2. Case 2: `C` <: CTT(`b`)
        - This is narrowing and requires run-time checks.
        - Consider `C` <: `B`:
            - If CTT(`b`) = `B` and RTT(`b`) = `C` (_or subtype of_ `C`), then it is allowed at run-time.
            - If CTT(`b`) = `B` and RTT(`b`) = `C` (_or other subtype of_ `B` _that is not_ `C`), then it not allowed at run-time.
        Since there is a _possibility_, the compiler will add codes to check at run-time.
    3. Case 3: `C` is an interface
        - Let RTT(`b`) = `B`.  Then it may have a subclass `A` such that `A` <: `C` (_i.e., implements the interface_ `C`).
        ```java
        class A extends B implements C { .. }
        ```
        - If RTT(`b`) = `A`, then it is allowed at run-time.

!!! failure "Impossibility"
    There are certain cases where it is _impossible_ for RTT(`b`) to be a subtype of `C`.  We will explain two cases here.

    1. Let CTT(`b`) = `B` and let both `B` and `C` be two unrelated classes (_.e.,_ `B` </: `C` _and_ `C` </: `B`).
        - Then it is impossible for RTT(`b`) to be subtype of `C` because the subclass of `B` must alreadt extends `B`.  As such, it cannot also extends from `C` as Java does not allow a class to extends from two or more classes.
    2. Let `C` be an interface and CTT(`b`) be a class with a modifier `final`.
        - Then Case (3) on the possibility does not apply as RTT(`b`) cannot be a subtype of CTT(`b`) since there cannot a subtype in the first place.  Recap: the `final` modifier on a class prevents the class from being inherited.

### Run-Time Check

The check at run-time is added on step (4) of compile-time check.  This is because at step (2) we are only looking for "_possibility_".  Therefore, there is a chance that such possibility did not occur at run-time.

1. Find the run-time type of variable `b` (_denoted_ RTT(`b`)).
2. Check if RTT(`b`) <: `C`.

You may think of the check for the type cast `(C) b` as logically equivalent to the following

```java
if (!(b instanceof C)) {
  throw new ClassCastException( .. );
}
```

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