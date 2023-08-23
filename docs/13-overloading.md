# Unit 13: Overloading

!!! abstract "Learning Objectives"

    Students should

    - understand what is overloading.
    - understand how to create overloaded methods.

## Method overloading

In the previous unit, we introduced _method overriding_. That is, when a subclass defines an instance method with the same _method descriptor_ as an instance method in the parent class.

In contrast, _method overloading_ is when we have two or more methods in the same class (_or inherited from superclass_) with the same name but a differing _method signature_[^1]. In other words, we create an overloaded method by changing the type, order, and number of parameters of the method but keeping the method name identical. 

[^1]: Note that this is not the same as the _method descriptor_. You can not overload a method by changing the return type.

Lets consider an `add` method which allows us to add two numbers, and returns the result. What if we would like to create an `add` method to sum up three numbers?

```Java
public int add(int x, int y) {
  return x + y;
}

public int add(int x, int y, int z) {
  return x + y + z;
}
```

In the example above, the methods `add(int, int)` and `add(int, int, int)` are overloaded. They have the same name but a different number of parameters. We can see that this allows us to write methods to handle differing inputs. 

Now lets consider our `Circle` class again. Our `Circle::contains(Point)` method allows us to check if a `Point` is within the radius of the current instance of the `Circle`. We would like to create a new method `Circle::contains(double, double)` which will allow us to check if an `x` and `y` coordinate (_another valid representation of a point_) is within our circle.

```Java
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
    return true;
    // TODO: Left as an exercise
  }
```

These two methods have the same method signature, and therefore `contains(double, double)` and `contains(double, double)` are not distinct methods. They are not overloaded, and therefore this above example will not compile.

```
_.java:_: error: method contains(double,double) is already defined in class Circle
  public boolean contains(double y, double x) {
                 ^
1 error
```

As it is also a method, it is possible to overload the class _constructor_ as well. As in the example below, we can see an overloaded constructor which gives us a handy way to instantiate a `Circle` object that is the unit circle centred at origin.

```Java
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

!!! info "Overriding vs Overloading"
    | Properties | Overriding | Overloading |
    |------------|------------|-------------|
    | Same method name | :material-check: | :material-check: |
    | Same method signature | :material-check: | :material-close: |
    | In the same class | :material-close: | :material-check: |
    | In the subclass | :material-check: | :material-check: |

    Note that the last point is quite subtle.  This is because the subclass _inherits_ all the method from the superclass.  As such, in the following example, we still have overloading.

    ```Java
    class T {
      void foo(double x) { }
    }

    class S extends T {
      void foo(int x, int y) { }
    }
    ```

## Chaining Constructor

If we look at the call to overloaded constructor `new Circle()`, it is equivalent to `new Circle(new Point(0, 0), 1)`.  Recap the spirit of abstraction

> _"Each significant piece of functionality in a program should be implemented in just one place in the source code. Where similar functions are carried out by distinct pieces of code, it is generally beneficial to combine them into one by abstracting out the varying parts."_

In the spirit of abstraction, we may want our overloaded constructor `Circle::Circle()` to invoke our original constructor `Circle::Circle(Point, double)`.  That way, any changes will have to be made in only one place.  This can be achieved by invoking `this(..)` as the first line on the constructor.  So, our `Circle` class can be written as follows.

```Java
class Circle {
  private Point c;
  private double r;

  public Circle(Point c, double r) {
    this.c = c;
    this.r = r;
  }

  // Overloaded constructor with a call to this(..)
  public Circle() {
    this(new Point(0, 0), 1);
  }
    :
}
```

If there are more constructors, we can have more chaining.  We may want to chain from the "simpler" constructor to the more "complex" constructor.  Consider adding another overloaded constructor that allows specifying a radius but will always be centred at origin.

```Java
class Circle {
  private Point c;
  private double r;

  public Circle(Point c, double r) {
    this.c = c;
    this.r = r;
  }

  // Overloaded constructor 1
  public Circle(double r) {
    this(new Point(0, 0), r); // chained to Circle::Circle(Point, double)
  }

  // Overloaded constructor 2
  public Circle() {
    this(1); // chained to Circle::Circle(double)
  }
    :
}
```

Remember how we mentioned that the call to `super(..)` (_if any_) in the constructor must be the first line in the constructor?  The call to `this(..)` (_if any_) must also be the first line in the constructor.  Therefore, you cannot have both call to `super(..)` and call to `this(..)`.  Which also means that if there is a call to `this(..)` default constructor is also not automatically added.