# Lab 04 Mini Problem

## Learning Objectives

- Students can create an immutable class

## Mini Problem 1
In this mini lab, we will create an immutable class. In object-oriented programming, _immutable object_ refers to an object whose state cannot be modified after it is created ([Wikipedia](https://en.wikipedia.org/wiki/Immutable_object)). You will learn more about immutability during lecture, but to motivate you for the time being, immutability makes it easy to reason about our code. Less changes == less bugs!

### Section 1: Creating an immutable class that contains primitive types
Let's start from a simple one: class that contains primitive types. Let's use `Point` from our previous lectures as an example. Suppose you have a point class defined as follows:
```java
class Point {
  public int x;
  public int y;

  public Point(int x, int y) {
    this.x = x;
    this.y = y;
  }

  void moveTo(int x, int y) {
    this.x = x;
    this.y = y;
  }

  @Override
  public String toString() {
    return "(" + this.x + "," + this.y + ")";
  }
}
```

Is it possible for the client to mutate the variables inside the `Point` class? Let's see some possible scenarios:
```java
Point p = new Point(0, 1);
p.x = 10;         // changes x!
p.moveTo(10, 15); // changes y!
```
**Task 1:** fix the point class so its state cannot be modified!

**Hint:** `moveTo` should return a `Point`.

Now, consider this sub-class of `Point`:
```java
class Point3D extends Point {
  public int z;

  public Point3D(int x, int y, int z) {
    super(x, y);
    this.z = z;
  }

  @Override
  Point3D moveTo(int x, int y, int z) {
    this.x = x;
    this.y = y;
    this.z = z;
    return this;
  }
}
```
**Task 2:** Do you think inheritance should be allowed? How can you fix this issue?

### Section 2: Creating an immutable class that contains arrays
To make things slightly more interesting, let's change the implementation of `Point`. Instead of using two integers `x` and `y`, let's use an array of integer:

```java
final class Point2 {
  private final int[] coord;

  public Point2(int[] coord) {
    this.coord = coord;
  }

  int[] getCoord() {
    return this.coord;
  }

  Point2 moveTo(int[] coord) {
    return new Point2(coord);
  }

  @Override
  public String toString() {
    return "(" + this.coord[0] + "," + this.coord[1] + ")";
  }
}
```

**Task 3:** Can you come up with a scenario that can cause the internal state of `Point2` to change? How to fix `Point2` such that it becomes immutable?

### Bonus: Creating an immutable class that contains mutable reference types
In our previous example, we see how an object is not truly immutable even when all the fields are `private final` and no setter methods are available. This is caused by the fact that the class contains one or more mutable object that can be modified from outside.

Consider this implementation of `Circle`:
```java
final class Circle {
  private final Point c;
  private final double r;

  public Circle (Point c, double r) {
    this.c = c;
    this.r = r;
  }

  public Circle moveTo(double x, double y) {
    return new Circle(c.moveTo(x, y), r);
  }
}
```
If `Point` is immutable, then `Circle` is also immutable. However, if `Point` is mutable, then `Circle` is also mutable.

**Bonus task:** Figure out how to make `Circle` class immutable even though `Point` is not immutable!