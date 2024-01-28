# Unit 16: Liskov Substitution Principle

!!! abstract "Learning Objectives"

    After this unit, the student should:

    - understand the type of bugs that reckless developers can introduce when using inheritance and polymorphism
    - understand the Liskov Substitution Principle and thus be aware that not all IS-A relationships should be modeled with inheritance
    - know how to explicitly disallow inheritance (for a class) or disallow overriding (for a method) with the `final` keyword

## The Responsibility When Using Inheritance

As you have seen in [Unit 14](14-polymorphism.md), polymorphism is a powerful tool that allows a client to change the behavior of existing code written by the implementer, behind the abstraction barrier.

As Ben Parker (aka Uncle Ben) said, "With great power, comes great responsibility."   The client must use overriding and inheritance carefully.  Since they can affect how existing code behaves, they can easily break existing code and introduce bugs.  Since the client may not have access to the existing code behind the abstraction barrier, it is often tricky to trace and debug.  Furthermore, the implementer would not appreciate it if their code was working perfectly until one day, someone overriding a method causes their code to fail, even without the implementer changing anything in their code.

Ensuring this responsibility cannot be done by the compiler, unfortunately.
It thus becomes a developer's responsibility to ensure that any inheritance with method overriding does not introduce bugs to existing code.  This brings us to the _Liskov Substitution Principle_ (LSP), which says: "Let $\phi(x)$ be a property provable about objects $x$ of type $T$. Then $\phi(y)$ should be true for objects $y$ of type $S$ where $S <: T$."

This is consistent with the definition of subtyping, $S <: T$, but spelled out more formally.

Let's consider the following example method, `Course::marksToGrade`, which takes in the marks of a student and returns the grade 'A', 'B', 'C', or 'F' as a `char`.  How `Course::marksToGrade` is implemented is not important.  Let's look at how it is used.

```Java
void displayGrade(Course c, double marks) {
  char grade = c.marksToGrade(marks);
  if (grade == 'A')) {
	System.out.println("well done");
  else if (grade == 'B') {
	System.out.println("good");
  else if (grade == 'C') {
	System.out.println("ok");
  } else {
	System.out.println("retake again");
  }
}
```

Now, suppose that one day, someone comes along and creates a new class `CSCUCourse` that inherits from `Course`, and overrides `marksToGrade` so that it now returns only 'S' and 'U'.  Since `CSCUCourse` is a subclass of `Course`, we can pass an instance to `displayGrade`:

```
displayGrade(new CSCUCourse("GEQ1000", 100));
```

and suddenly `displayGrade` is displaying `retake again` even if the student is scoring 100 marks.

The example above shows that we are violating the LSP unintentionally. The object `m` has the following property: `m.marksToGrade` always returns something from the set { `'A'`, `'B'`, `'C'`, `'F'` }, that the method `displayGrade` depends on explicitly.  The subclass `CSCUCourse` violated that and made `m.marksToGrade` return `'S'` or `'U'`, sabotaging `displayGrade` and causing it to fail.

LSP cannot be enforced by the compiler[^1]. The properties of an object have to be managed and agreed upon among programmers.  A common way is to document these properties as part of the code documentation.

[^1]: We can use `assert` to check some of the properties though.

## LSP Through the Lens of Testing

Another way to develop an intuition of the LSP is through the lens of testing. When we write a method, we may want to introduce test cases to check that our method is working correctly. These test cases are designed based on the _specification_ of our method and not its implementation details[^2]. That is, we test based on the expected inputs and resultant outputs.

[^2]: The test cases we are describing here are known as black-box tests and you will encounter these in later courses at NUS. We will not go into any further details in this course.

Let's look at an example. We would like to model a restaurant booking system for a restaurant chain. Consider the following `Restaurant` class.  Every restaurant in the chain opens at 12 PM and closes at 10 PM, and has a singular method `canMakeReservation` which allows us to check if the restaurant is available for reservations at a certain `time`.  **The requirement given is that the system must be able to process a reservation during its opening hours.**

```Java
public class Restaurant {
  public static final int OPENING_HOUR = 1200;
  public static final int CLOSING_HOUR = 2200;

  public boolean canMakeReservation(int time) {
    if (time <= CLOSING_HOUR && time >= OPENING_HOUR) {
      return true;
    }
    return false;
  }
}
```

The method `canMakeReservation` returns `true` when the argument passed into `time` is between 12 PM and 10 PM. Let's think about how we would test this method.  Two important edge cases to test are to check if the method returns true for the stated restaurant opening and closing hours.

```Java
Restaurant r = new Restaurant();
r.canMakeReservation(1200) == true; // Is true, therefore test passes
  : // test for other hours between 1200 - 2200
r.canMakeReservation(2200) == true; // Is true, therefore test passes
```

Note that these are simple `jshell` tests, in software engineering courses you will learn better ways to design and formalize these tests.

We can now rephrase our LSP in terms of testing. A _subclass_ should not break the expectations set by the _superclass_. If a class `B` is substitutable for a parent class `A` then it should be able to pass all test cases of the parent class `A`. If it does not, then it is not substitutable and the LSP is violated.

Let's now consider two subclasses of `Restaurant`, `LunchRestaurant` and `DigitalReadyRestaurant`. Our `LunchRestaurant` does not take reservations during peak hours (12 to 2 pm).

```Java
public class LunchRestaurant extends Restaurant {
  private final int peakHourStart = 1200;
  private final int peakHourEnd = 1400;

  @Override
  public boolean canMakeReservation(int time) {
    if (time <= peakHourEnd && time >= peakHourStart) {
      return false;
    } else if (time <= CLOSING_HOUR && time >= OPENING_HOUR) {
      return true;
    }
    return false;
  }
}
```

`LunchRestaurant` does not take reservations during peak hours (i.e., 1200 to 1400). As `LunchRestaurant` $<:$ `Restaurant`, we can point our variable `r` to a new instance of `LunchRestaurant` and run the test cases of the parent class, as can be seen in the code below.

```Java
Restaurant r = new LunchRestaurant();
r.canMakeReservation(1200) == true; // Is false, therefore test fails
  : // test for other hours between 1200 - 2200
r.canMakeReservation(2200) == true; // Is true, therefore test passes
```

Whilst the second test passes, the first test does not since it falls within the peak lunch hour.  Therefore `LunchRestaurant` is not substitutable for `Restaurant` and the LSP is violated.  We have changed the expectation of the method in the child class.

Let's suppose the restaurant chain starts to roll out an online reservation system for a subset of its restaurants.  These restaurants can take reservations at any time.
We create a subclass `DigitalReadyRestaurant`, as follows:

```Java
public class DigitalReadyRestaurant extends Restaurant {

  @Override
  public boolean canMakeReservation(int time) {
    return true;
  }
}
```

Similarly, as `DigitalReadyRestaurant` $<:$ `Restaurant`, we can point our variable `r` to a new instance of `DigitalReadyRestaurant` and run the test cases of the parent class, as can be seen in the code below.

```Java
Restaurant r = new DigitalReadyRestaurant();
r.canMakeReservation(1200) == true; // Is true, therefore test passes
  : // test for other hours between 1200 - 2200
r.canMakeReservation(2200) == true; // Is true, therefore test passes
```

Both test cases pass.  In fact, all test cases that pass for `Restaurant` would pass for `DigitalReadyRestaurant`.  Therefore `DigitalReadyRestaurant` is substitutable for `Restaurant`. Anywhere we can use an object of type `Restaurant`, we can use `DigitalReadyRestaurant` without breaking any previously written code.

We can now rephrase our LSP in terms of testing. A _subclass_ should not break the expectations set by the _superclass_. If a class `B` is substitutable for a parent class `A` then it should be able to pass all test cases of the parent class `A`. If it does not, then it is not substitutable and the LSP is violated.

## Preventing Inheritance and Method Overriding

Sometimes, it is useful for a developer to explicitly prevent a class from being inherited.  Not allowing inheritance would make it much easier to argue for the correctness of programs, something that is important when it comes to writing secure programs.  The two Java classes you have seen, `java.lang.Math` and `java.lang.String`, cannot be inherited from.  In Java, we use the keyword `final` when declaring a class to tell Java that we ban this class from being inherited.

For example, to prevent `Circle` from being inherited,
```Java
final class Circle {
    :
}
```

Alternatively, we can allow inheritance but still prevent a specific method from being overridden, by declaring a method as `final`.  Usually, we do this on methods that are critical for the correctness of the class.

For instance, to prevent `contains` from being overridden,
```Java
class Circle {
    :
  public final boolean contains(Point p) {
      :
  }
}
```

We have now seen that the `final` keyword can be used in three places:

1. In a class declaration to prevent inheritance.
2. In a method declaration to prevent overriding.
3. In a field declaration to prevent re-assignment.
