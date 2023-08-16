# Unit 6: Tell, Don't Ask

!!! abstract "Learning Objectives"

    Students should

    - understand what accessor and mutator are used for, and why not to use them.
    - understand the principle of "Tell, Don't Ask".

## Accessors and Mutators

Similar to providing constructors, a class should also provide methods to retrieve or modify the properties of the object.

- **Accessors**: Methods that retrieve the properties of an object (_i.e., retrieve the value of a field_).  Also known as _getter_.
- **Mutators**: Methods that modify the properties of an object (_i.e., update the value of a field_).  Also known as _setter_.

The use of **both** accessor and mutator methods is a bit controversial.   Suppose that we provide an accessor method and a mutator method for every private field, then we are exposing the internal representation, therefore breaking the encapsulation.  For instance:

```Java
// Circle v0.4
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

Do note that the name `getProp` or `setProp` for a field `prop` is _optional_.  But it is a good practice.  For instance, we may actually name the accessor for `r` as `getRadius` and the mutator as `setRadius` shown below.

```Java
  public double getRadius() {       // originally getR
    return this.r;
  }

  public void setRadius(double r) { // originally setR
    this.r = r;
  }
```

!!! danger "Both Accessor and Mutator"
    As we have said above, having both accessor and mutator for a private field is controversial.  But it is still _better_ than setting the field itself public.  By having accessor and mutator, we are still adding some layer of abstraction because we can change the name of the field without affecting the client.

    Another advantage is that we may be able to perform some checks on the mutator and prevents certain invalid values from ever being assigned to the field.  Consider the method `setR` in our circle v0.4 above.  A slightly better approach is to implement it with a check to prevent setting the radius zero or negative.

    ```java
    public void setR(double r) {
      if (r > 0) {
        this.r = r;
      }
    }
    ```

    Still, you should first check if you really need an accessor or a mutator for all fields.

## The "Tell Don't Ask" Principle

The mutators and accessors above are pretty pointless.  If we need to know the internal and do something with it, then we are breaking the abstraction barrier.  However, this is still _slightly_ better than accessing the field directly.  Using accessors and mutators at least allow the implementer to change the name of the fields.  Unfortunately, the implementer cannot change the name of the accessors or mutators.

!!! example "Possible Change #1"
    ```Java
    class Circle {
      private double x;
      private double y;
      private double radius;

      public Circle(double x, double y, double r) {
        this.x = x;
        this.y = y;
        this.radius = r;
      }

      // no change to getX, setX, getY, and setY

      public double getR() {
        return this.radius;
      }

      public void setR(double r) {
        this.radius = r;
      }

      public double getArea() {
        return 3.141592653589793 * this.r * this.r;
      }
    }
    ```

So in a way, the changes are more _localized_.  Here, the localization is only within the class `Circle`.  The client of `Circle` need not change their code.  However, this is still not the best approach.  The better approach is to implement a method within the class that does whatever we want the class to do.

For instance, suppose that we want to check if a given point (_x,y_) falls within the circle, one approach would be:

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

This better approach involves writing a few more lines of code to implement the method, but it keeps the encapsulation intact.  If one fine day, the implementer of `Circle` decided to change the representation of the circle and remove the direct accessors to the fields, then only the implementer needs to change the implementation of `contains`.  The client does not have to change anything.

!!! example "Possible Change #2"
    ```Java
    class Circle {
      private double t; // store the top-left corner of
      private double l; // the bounding square instead
      private double r;

      public Circle(double x, double y, double r) {
        this.t = y + r; // compute the top bound
        this.l = x - r; // compute the left bound
        this.r = r;
      }

      // no more accessors/mutators

      public double getArea() {
        return 3.141592653589793 * this.r * this.r;
      }

      public boolean contains(double x, double y) {
          : // implementation omitted but changed!
      }
    }
    ```

The principle around which we can think about this is the "Tell, Don't Ask" principle.  The client should tell a `Circle` object what to do (_compute the circumference_), instead of asking "what is your radius?" to get the value of a field then perform the computation on the object's behalf.

Note that in both possible changes above, the client are unaffected.  Further note that in the second possible change -- assuming the client does not use any accessors or mutators -- the code written by the client has more resistance to changes.  We not only change the name of the field but also the method implementation by redefining what it means to be contained.  The client may not even realized that internally, the implementation of `Circle` has been modified!

!!! notes "Tips and Tricks"

    The first tips is related to information hiding.  Although there are cases where we really have to make certain fields `public`, we would like to -- as much as possible -- first set **all** the fields as `private`.  Afterwards, we will need to think about the methods that are necessary for the class.  This is where the next tips will be useful.

    While there are situations where we can't avoid using accessor or modifier in a class, for beginner OO programmers like yourself, it is better to not define classes with any accessor and modifier to the private fields, and forces yourselves to think in the OO way -- to tell an object what task to perform as a client, and then implement this task within the class as a method as the implementer.

    One way to think about this is to try to think about the _responsibility_ of the class.  Recap that the class names are typically noun.  So think about the actual object and imagine what is their responsibility.

    Take for instance a class called `Ball` and a class called `Player` (_i.e., soccer player_).  We can imagine that the player can _kick_ a ball.  This is captured by the method `void kick(Ball b, double speed)` in the class `Player`.  However, it is the responsibility of the `Ball` to keep track of its own position.  This might be done by the method `void move(double speed)`.

    More closely related to the circle example, we may look at some alternative choices.  Say, for instance, we want to compute the square of the distance from the center point of the circle to any other (_x, y_) coordinates.  We may implement the following method

    ```java
    public double distanceSquared(double x, double y) {
      double dx = this.x - x;
      double dy = this.y - y;
      return dx * dx + dy * dy;
    }
    ```

    Given this method, we may not even need to provide the method `contains`.  Instead, we may choose to provide __both__ `distanceSquared` and `getRadius`.  The latter is needed because we can check if a point (_x, y_) is contained within a circle if the square of the distance to the center point is less than or equal to the square of the radius.

    So which design to choose?  Typically we want to minimize the number of accessors and mutators that we have.  So the design using `contains` is preferred.  If you cannot come up with such design then having both `distanceSquared` and `getRadius` is still preferable to having accessors for all three fields.

## Further Reading

- [Tell Don't Ask](https://martinfowler.com/bliki/TellDontAsk.html) by Martin Fowler
- [Why getters and setters are evil](https://www.infoworld.com/article/2073723/why-getter-and-setter-methods-are-evil.html), by Allen Holub, JavaWorld
- [Getters and setters are evil. Period](https://www.yegor256.com/2014/09/16/getters-and-setters-are-evil.html), by Yegor Bygayenko.
