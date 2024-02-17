# Unit 6: Tell, Don't Ask

!!! abstract "Learning Objectives"

    After this unit, students should:

    - understand what accessor and mutator are used for, and why not to use them
    - understand the principle of "Tell, Don't Ask"

## Accessors and Mutators

Similar to providing constructors, a class should also provide methods to retrieve or modify the properties of the object.  These methods are called the _accessor_ (or _getter_) or _mutator_ (or _setter_).

The use of accessor and mutator methods is a bit controversial.   Suppose that we provide an accessor method and a mutator method for every private field, then we are exposing the internal representation, therefore breaking the encapsulation.  For instance:

```Java title="Circle v0.4"
class Circle {
  private double x;
  private double y;
  private double r;

  public Circle(double x, double y, double r) {
    this.x = x;
    this.y = y;
    this.r = r;
  }

  public double getX() {
    return this.x;
  }

  public void setX(double x) {
    this.x = x;
  }

  public double getY() {
    return this.y;
  }

  public void setY(double y) {
    this.y = y;
  }

  public double getR() {
    return this.r;
  }

  public void setR(double r) {
    this.r = r;
  }

  public double getArea() {
    return 3.141592653589793 * this.r * this.r;
  }
}
```

In the code above, we can categorise the accessor and mutator of each field as follows.

| Fields | Accessors | Mutators |
|--------|-----------|----------|
| `x` | `getX` | `setX` |
| `y` | `getY` | `setY` |
| `r` | `getR` | `setR` |

!!! danger "Why provide an accessor and a mutator when you can declare a field as `public`?"
    Whilst having both an accessor and a mutator for a private field is controversial, it is still _better_ than setting the field `public`.  By having an accessor and a mutator, we are adding a layer of abstraction.  For instance, we can still rename a field without affecting the client.

    Another advantage is that we may be able to perform some checks on the mutator and prevent certain invalid values from ever being assigned to the field.  Consider the method `setR` in our `Circle` v0.4 above.  A slightly better approach is to implement it with a check to prevent setting the radius to a non-positive value.

    ```java
    public void setR(double r) {
      if (r > 0) {
        this.r = r;
      } else {
        // handle error
      }
    }
    ```

    Regardless, we should think carefully if an accessor or a mutator is really needed for a field.

## The "Tell, Don't Ask" Principle

The mutators and accessors above are pretty pointless.  If we need to know the internal and do something with it, then we are breaking the abstraction barrier.  The right approach is to implement a method within the class that does whatever we want the class to do.   For instance, suppose that we want to check if a given point (x,y) falls within the circle, one approach would be:

```Java
double cX = c.getX();
double cY = c.getY();
double r = c.getR();
boolean isInCircle = ((x - cX) * (x - cX) + (y - cY) * (y - cY)) <= r * r;
```

where `c` is a `Circle` object.

A better approach would be to add a new `boolean` method in the `Circle` class, and call it instead:
```Java
boolean isInCircle = c.contains(x, y);
```

This better approach involves writing a few more lines of code to implement the method, but it keeps the encapsulation intact.  If one fine day, the implementer of `Circle` decides to change the representation of the circle and remove the direct accessors to the fields, then only the implementer needs to change the implementation of `contains`.  The client does not have to change anything.

The principle around which we can think about this is the "Tell, Don't Ask" principle.  The client should _tell_ a `Circle` object what to do (compute the circumference), instead of _asking_ "What is your radius?" to get the value of a field, then perform the computation on the object's behalf.

While there are situations where we cannot avoid using an accessor or a mutator in a class, for beginner OO programmers like yourself, it is better to not define classes with any accessor and modifier to the private fields and force yourselves to think in the OO way &mdash; to tell an object what task to perform as a client, and then implement this task within the class as a method as the implementer.

## Further Reading

- [Tell Don't Ask](https://martinfowler.com/bliki/TellDontAsk.html) by Martin Fowler
- [Why getters and setters are evil](https://www.infoworld.com/article/2073723/why-getter-and-setter-methods-are-evil.html), by Allen Holub, JavaWorld
- [Getters and setters are evil. Period](https://www.yegor256.com/2014/09/16/getters-and-setters-are-evil.html), by Yegor Bygayenko.
