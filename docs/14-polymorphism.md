# Unit 14: Polymorphism

!!! abstract "Learning Objectives"

    Students should

    - understand polymorphism.
    - be aware of dynamic binding.
    - be aware of the `equals` method and the need to override it to customize the equality test.
    - understand when narrowing type conversion and type casting are allowed.

## Taking on Many Forms

Method overriding enables _polymorphism_, the fourth and the last pillar of OOP.  Arguably this is the most powerful pillar.  It allows us to change how existing code behaves, without changing a single line of the existing code (_or even having access to the code_).

Consider the function `say(Object)` below:
```Java
void say(Object obj) {
  System.out.println("Hi, I am " + obj.toString());
}
```

Note that this method receives an `Object` instance.  Since both `Point` <: `Object` and `Circle` <: `Object`, we can do the following (_recap: "whenever a superclass is needed, a subclass can be given"_):
```Java
Point p = new Point(0, 0);
say(p);
Circle c = new Circle(p, 4);
say(c);
```

When executed, `say` will first print `Hi, I am (0.0, 0.0)`, followed by `Hi, I am { center: (0.0, 0.0), radius: 4.0 }`.  _We are invoking the overriding_ `Point::toString()` _in the first call, and_ `Circle::toString()` _in the second call_.  The same method invocation `obj.toString()` causes two different methods to be called in two separate invocations!

In biology, polymorphism means that an organism can have many different forms.  Here, the variable `obj` can have many forms as well.  Which method is invoked is decided _during run-time_, depending on the run-time type of the `obj`.  This is called _dynamic binding_ or _late binding_ or _dynamic dispatch_.

Before we get into this in more detail, let's consider overriding `Object::equals(Object)`.

## The `equals` method

`Object::equals(Object)` compares if two object references refer to the same object.  That is also the standard behavior of the `==` operator.  In the case of `==` operator, we can say that it compares the _value_.  So what is the _value_ of a reference type?  Well, it must be the reference (_i.e., the address_).

!!! info "`equals` Method"
    While it is not necessary to know this, `equals` method should satisfy an _equivalence relation_ on non-null object references:

    - It is _reflexive_: for any non-null reference value `x`, `x.equals(x)` should return `true`.
    - It is _symmetric_: for any non-null reference values `x` and `y`, `x.equals(y)` should return `true` if and only if `y.equals(x)` returns `true`.
    - It is _transitive_: for any non-null reference values `x`, `y`, and `z`, if `x.equals(y)` returns `true` and `y.equals(z)` returns `true`, then `x.equals(z)` should return `true`.
    - It is _consistent_: for any non-null reference values `x` and `y`, multiple invocations of `x.equals(y)` consistently return `true` or consistently return `false`, provided no information used in equals comparisons on the objects is modified.
    - For any non-null reference value `x`, `x.equals(null)` should return `false`.

    One thing to be careful of when using `equals` method is that you must ensure in the method invocation `obj.equals(arg)`, the value of `obj` is not `null`.

Suppose we have:

```Java
Circle c0 = new Circle(new Point(0, 0), 10);
Circle c1 = new Circle(new Point(0, 0), 10);
Circle c2 = c1;
```

`c2.equals(c1)` returns `true`, but `c0.equals(c1)` returns `false`.  Even though `c0` and `c1` are _semantically_ the same, they refer to the two different objects.

To compare if two circles are _semantically_ the same, we need to override this method[^1].  After all, Java does not know what you meant by having two circle being equal.  There are other possibilities including (_but not limited to_)

- having equal radius
- having equal center point
- having both equal radius and center point

[^1]: If we override `equals(Object)`, we should generally override `hashCode()` as well, but let's leave that for another lesson on another day.

```Java hl_lines="40-50"
// version 0.7
/**
 * A Circle object encapsulates a circle on a 2D plane.  
 */
class Circle {
  private Point c;   // the center
  private double r;  // the length of the radius

  /**
   * Create a circle centered on Point c with given radius r
   */
  public Circle(Point c, double r) {
    this.c = c;
    this.r = r;
  }

  /**
   * Return the area of the circle.
   */
  public double getArea() {
    return Math.PI * this.r * this.r;
  }

  /**
   * Return true if the given point p is within the circle.
   */
  public boolean contains(Point p) {
    return false;
    // TODO: Left as an exercise
  }

  /**
   * Return the string representation of this circle.
   */
  @Override
  public String toString() {
    return "{ center: " + this.c + ", radius: " + this.r + " }";
  }

  /**
   * Return true the object is the same circle (i.e., same center, same radius).
   */
  @Override
  public boolean equals(Object obj) {
    if (obj instanceof Circle) {
      Circle circle = (Circle) obj;
      return (circle.c.equals(this.c) && circle.r == this.r);
    }
    return false;
  }
}
```

This is more complicated than `toString`.  There are a few new concepts involved here so let's go through them slowly.  Click on the tabs below to find out more.

=== "`instanceof` Operator"
    ```Java hl_lines="2"
    public boolean equals(Object obj) {
      if (obj instanceof Circle) {
        Circle circle = (Circle) obj;
        return (circle.c.equals(this.c) && circle.r == this.r);
      }
      return false;
    }
    ```

    `equals` takes in a parameter of compile-time type `Object` (_Line 1_).  It only makes sense if we compare (_during run-time_) a circle with another circle.  So, we first check if the run-time type of `obj` is a subtype of `Circle`.  This is done using the `instanceof` operator.  The operator returns `true` if `obj` has a run-time type that is a subtype of `Circle`.

    ---

=== "Fields Access"
    ```Java hl_lines="3 4"
    public boolean equals(Object obj) {
      if (obj instanceof Circle) {
        Circle circle = (Circle) obj;
        return (circle.c.equals(this.c) && circle.r == this.r);
      }
      return false;
    }
    ```

    To compare `this` circle with the given circle, we have to access the center `c` and radius `r`.  But if we access `obj.c` or `obj.r`, the compiler will complain.  As far as the compiler is concerned, `obj` has the compile-time type `Object`, and there is no such fields `c` and `r` in the class `Object`!  This is why, after assuring that the run-time type of `obj` is a subtype of `Circle`, we assign `obj` to another variable `circle` that has the compile-time type `Circle`.  We finally check if the two centers are equal (_again,_ `Point::equals` _is left as an exercise_) and the two radii are equal[^2].

    Another important point is that `circle.c` and `circle.r` are allowed even when the fields `c` and `r` are declared `private` because we are inside the class `Circle`.  So even if the fields belong to another object, as long as it is in the same class, it is allowed.

    ---

=== "Explicit Type Casting"
    ```Java hl_lines="3"
    public boolean equals(Object obj) {
      if (obj instanceof Circle) {
        Circle circle = (Circle) obj;
        return (circle.c.equals(this.c) && circle.r == this.r);
      }
      return false;
    }
    ```

    The statement that assigns `obj` to `circle` involves _type casting_.  We mentioned before that Java is strongly typed and so it is very strict about type conversion.  Here, Java allows type casting from type $T$ to $S$ if $S <: T$[^3].  By allow, we meant that Java compiler will not give compilation error.

    This is called _narrowing type conversion_.  Unlike widening type conversion, which is always allowed and always correct, a _narrowing type conversion_ requires explicit typecasting and validation during run-time.  If we do not ensure that `obj` has the correct run-time type, casting can lead to a run-time error (_which if you [recall](01-compiler.md), is bad_).

    The syntax for explicit type conversion is `(Type) expr`.  Java will attempt to convert the result of the expression `expr` into the type `Type`.  While you may also add explicit type casting for _widening_ type conversion, such explicit type casting is not necessary and can be omitted.

    ---

[^2]: The right way to compare two floating-point numbers is to take their absolute difference and check if the difference is small enough.  We are sloppy here to keep the already complicated code a bit simpler.  You shouldn't do this in your code.

[^3]: This is not the only condition where type casting is allowed. We will look at other conditions in later units.

All these complications would go away, however, if we define `Circle::equals` to take in a `Circle` as a parameter, like this:

```Java
class Circle {
    :
  /**
   * Return true the object is the same circle (i.e., same center, same radius).
   */
  @Override
  public boolean equals(Circle circle) {
    return (circle.c.equals(this.c) && circle.r == this.r);
  }
}
```

This version of `equals` however, does not override `Object::equals(Object)`.  Since we hinted to the compiler that we meant this to be an overriding method, using `@Override`, the compiler will give us an error.  This is not treated as method overriding, since the signature for `Circle::equals(Circle)` is different from `Object::equals(Object)`.

Why then is overriding important?  Why not just leave out the line `@Override` and live with the non-overriding, one-line, `equals` method above?  Keep this question in mind as this will be answered when we discuss dynamic binding.

!!! info "Why is it called Narrowing?"
    Because the subclass inherits from superclass, it may seem like the subclass has "more" properties.  So it seems counter-intuitive why such conversion is called narrowing.  Take, for instance, `Circle` and `ColoredCircle`.  `ColoredCircle` has an additional property called `color`.  So why converting from `Circle` to `ColoredCircle` is called narrowing?

    The reason here is that a `ColoredCircle` **_is a_** `Circle` but the opposite is not true (_i.e.,_ `Circle` _is **not**_ `ColoredCircle`).  Consider adding another class `SpinningCircle` such that `SpinningCircle` <: `Circle`.  Then, `Circle` contains all possible `Circle` + all possible `ColoredCircle` + all possible `SpinningCircle`.  On the other hand, `SpinningCircle` only contains all possible `SpinningCircle`.

    We can see that the possible values for `Circle` is larger than the possible values for `SpinningCircler`.  So converting from `Circle` to `SpinningCircle` will indeed narrow down our possible value.

## The Power of Polymorphism

Let's consider the following example.  Suppose we have a general `contains` method that takes in an array of objects.  The array can store any type of objects: `Circle`, `Square`, `Rectangle`, `Point`, `String`, etc.  The method `contains` also takes in a target `obj` to search for, and returns true if there is an object in `array` that equals to `obj`.

```Java hl_lines="4"
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

With overriding and polymorphism, the magic happens in Line 4 -- depending on the **_run-time_** type of `curr`, the corresponding, customized version of `equals` is called to compare against `obj`.  So if the run-time type of `curr` is `Circle`, then we will invoke `Circle::equals(Object)` and if the run-time type of `curr` is `Point`, then we will invoke `Point::equals(Object)`.  This, of course, assumes that `Object::equals(Object)` is overridden in both classes.

Now, if `Circle::equals` takes in a `Circle` as the parameter (_i.e.,_ `Circle::equals(Circle)`), the call to `equals` inside the method `contains` would not invoke `Circle::equals(Circle)`.  It would invoke `Object::equals(Object)` instead due to the matching method signature, and we can't search for `Circle` based on semantic equality.

But why are we searching for `equals(Object)` in the first place?  Look closely at how the method is invoked: `curr.equals(obj)`.  Here, we can see that the parameter we are passing is `obj`.  The **_compile-time_** type of `obj` is `Object` as seen from the parameter declaration at Line 2.  So at compile-time, we only know that its type is `Object`.

To have a generic `contains` method without polymorphism and overriding, we will have to do something like this:

```Java
// version 0.2 (without polymorphism)
boolean contains(Object[] array, Object obj) {
  for (Object curr : array) {
    if (obj instanceof Circle) {
      if (curr.equals((Circle)obj)) {
        return true;
      }
    } else if (obj instanceof Square) {
      if (curr.equals((Square)obj)) {
        return true;
      }
    } else if (obj instanceof Point) {
      if (curr.equals((Point)obj)) {
        return true;
      }
    }
     :
  }
  return false;
}
```

which is not scalable since every time we add a new class, we have to come back to this method and add a new branch to the `if-else` statement!

As this example has shown, polymorphism allows us _to write succinct code that is future proof_.  By dynamically deciding which method implementation to execute during run-time, the implementer can write short yet very general code that works for existing classes as well as new classes that might be added in the future by the client, without even the need to re-compile!

!!! info "Java Array"
    Note that there are two ways to declare an array in Java:

    1. `Type[] var`
    2. `Type var[]` (_i.e., C-style array_)

    Please follow the first style as it will be less confusing.  That is also the preferred style based on our style guide.

    The next point will be important later when we talk about variance of types.  When we declare a class in Java like the class `A` below:

    ```java
    class A { .. }
    ```

    Another class is also created corresponding to the type of _array of type_ `A`.  The name given by Java internally is `[LA;`.  Notice how the name is not a valid name in Java, so you cannot create such a class by yourself.  This class has a field called `length` which is the number of element the array instance may contain.  This should hopefully explains why to get the number of elements in an array, we use `arr.length` and not `arr.length()` because it is a field and not a method.

    ```java
    class [LA; { .. } // but not a legal name
    ```

    The more interesting thing is when you declare a subtype of class `A` such as the class `B` below:

    ```java
    class B extends A { .. }
    ```

    The type of _array of type_ `B` (_i.e.,_ `[LB;`) is automatically created such that `[LB;` is a subtype of `[LA;`.  In other words,

    ```java
    class [LB; extends [LA; { .. } // still not a legal name
    ```

## Adding Class

So to recap, one of the main benefit of polymorphism is the ability to extend your current program with new classes without without modifying what you have written before.  To make the example more explicit, imagine that the following classes are already written.

```Java
class Animal {
  public String name() {
    return "Animal";
  }

  public String sound() {
    return "Grrrr";
  }
}

class Cow extends Animal {
  @Override
  public String name() {
    return "Cow";
  }

  @Override
  public String sound() {
    return "Moo";
  }
}

class Duck extends Animal {
  @Override
  public String name() {
    return "Duck";
  }

  @Override
  public String sound() {
    return "Quack";
  }
}
```

Consider the following method to sing Old MacDonald.

```Java
void sing(Animal[] animals) {
  for(Animal animal : animals) {
    System.out.println("Old MacDonald had a farm E-I-E-I-O");
    System.out.println("And on that farm he had a " + animal.name() + " E-I-E-I-O");
    System.out.println("With a " + animal.sound() + " " + animal.sound() + " here");
    System.out.println("and a " + animal.sound() + " " + animal.sound() + " there");
    System.out.println("Here a " + animal.sound() + " there a " + animal.sound());
    System.out.println("Everywhere a " + animal.sound() + " " + animal.sound());
    System.out.println("Old MacDonald had a farm E-I-E-I-O");
  }
}
```

Now if we want to add a new animal, we simply have to create a new class that extends the `Animal` class and implements the two methods `name` and `sound`.  We do not have to modify the all the codes we have written before.

```Java
class Dog extends Animal {
  @Override
  public String name() {
    return "Dog";
  }

  @Override
  public String sound() {
    return "Woof";
  }
}
```

Notice how the method `sing(Animal[])` works without modification.  Additionally, we do not have to modify all the other classes as well.  That is the power of polymorphism.

## Adding Functionality

There is a "dual" problem to adding a class and that's adding functionality.  This problem is difficult in Java.  Consider the animals above including `Animal`, `Cow`, `Duck`, and `Dog`.  Now since all animals can move but they move differently, let's add a method called `move()`.  Where should we add this method to?

Firstly, since all animals can move, we have to add this method to the `Animal` class.  Secondly, since all animals move differently, we also have to override this method in `Cow`, `Duck`, `Dog`.  Therefore, the act of adding a single functionality requires us to modify all the relevant classes!

In this case, we only have 4 classes, but if we have more classes, there are more places we have to changed.  This breaks the principle of abstraction.  In fact, this breaks abstraction barrier if we are not the implementer of one of the animal classes.  To make matters worse, if we forgot to override this method in one of the subclasses, we will not even get compilation error.

!!! info "Design"
    Hopefully you understand the power of polymorphism as well as its limitation.  In particular, when you design your classes according to OOP principle, you should try to design it in such a way that the changes you expect are about adding classes rather than adding functionality.