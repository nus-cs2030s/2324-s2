# Unit 13: Overloading

!!! abstract "Learning Objectives"

    After reading this unit, students should

    - understand what is overloading
    - understand how to create overloaded methods

## Method overloading

In the previous unit, we introduced _method overriding_ -- when a subclass defines an instance method with the same _method descriptor_ as an instance method in the parent class.

In contrast, _method overloading_ is when we have two or more methods in the same class with the same name but a differing _method signature_[^1]. In other words, we create an overloaded method by changing the type, order, and number of parameters of the method but keeping the method name identical. 

[^1]: Note that this is not the same as the _method descriptor_. You can not overload a method by changing the return type.

Let's consider an `add` method which allows us to add two numbers, and returns the result. What if we would like to create an `add` method to sum up three numbers?

```Java
public int add(int x, int y) {
  return x + y;
}

public int add(int x, int y, int z) {
  return x + y + z;
}
```

In the example above, the methods `add(int, int)` and `add(int, int, int)` are overloaded. They have the same name but a different number of parameters. We can see that this allows us to write methods to handle differing inputs. 

Now let's consider our `Circle` class again. Our `Circle::contains(Point)` method allows us to check if a `Point` is within the radius of the current instance of the `Circle`. We would like to create a new method `Circle::contains(double, double)` which will allow us to check if an (`x`, `y`) coordinate (another valid representation of a point) is within our circle.

```Java title="Circle with Overloading `contains` Method"
import java.lang.Math;

class Circle {
  private Point c;   
  private double r;  

  public Circle(Point c, double r) {
    this.c = c;
    this.r = r;
  }

  public double getArea() {
    return Math.PI * this.r * this.r;
  }

  public boolean contains(Point p) {
    return false;
    // TODO: Left as an exercise
  }

  public boolean contains(double x, double y) {
    return false;
    // TODO: Left as an exercise
  }

  @Override
  public String toString() {
    return "{ center: " + this.c + ", radius: " + this.r + " }";
  }
}
```
In the above example, `Circle::contains(Point)` and `Circle::contains(double, double)` are overloaded methods. 

Recall that overloading requires changing the order, number, and/or type of parameters and says nothing about the names of the parameters. Consider the example below, where we have two `contains` methods in which we swap parameter names. 

```Java
  public boolean contains(double x, double y) {
    return false;
    // TODO: Left as an exercise
  }

  public boolean contains(double y, double x) {
    return false;
    // TODO: Left as an exercise
  }
```

These two methods have the same method signature, and therefore `contains(double, double)` and `contains(double, double)` are not distinct methods. They are not overloaded, and therefore this example above will not compile.

As it is also a method, it is possible to overload the class _constructor_ as well. As in the example below, we can see an overloaded constructor which gives us a handy way to instantiate a `Circle` object that is the unit circle.

```Java title="Circle with Overloaded Constructor"
class Circle {
  private Point c; 
  private double r;

  public Circle(Point c, double r) {
    this.c = c;
    this.r = r;
  }

  // Overloaded constructor
  public Circle() {
    this.c = new Point(0, 0);
    this.r = 1;
  }
  :
}
```

```Java
// c1 points to a new Circle object with a centre (1, 1) and a radius of 2
Circle c1 = new Circle(new Point(1, 1), 2); 
// c2 points to a new Circle object with a centre (0, 0) and a radius of 1
Circle c2 = new Circle();
```

It is also possible to overload `static` class methods in the same way as instance methods. In the next unit, we will see how Java chooses which method implementation to execute when a method is invoked.