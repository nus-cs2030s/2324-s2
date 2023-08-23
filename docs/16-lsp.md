# Unit 16: Liskov Substitution Principle

!!! abstract "Learning Objectives"

    Students should

    - understand the type of bugs that reckless developers can introduce when using inheritance and polymorphism.
    - understand the Liskov Substitution Principle and thus be aware that not all IS-A relationships should be modeled with inheritance.
    - know how to explicitly disallow inheritance when writing a class or disallow overriding with the `final` keyword.

## The Responsibility When Using Inheritance

As you have seen in [Unit 14](14-polymorphism.md), polymorphism is a powerful tool that allows a client to change the behavior of existing code written by the implementer, behind the abstraction barrier.

As Ben Parker (_aka Uncle Ben_) said, "With great power, comes great responsibility."   The client must use overriding and inheritance carefully.  Since they can affect how existing code behaves, they can easily break existing code and introduce bugs.  Since the client may not have access to the existing code behind the abstraction barrier, it is often tricky to trace and debug.  Furthermore, the implementer would not appreciate it if their code was working perfectly until one day, someone overriding a method causes their code to fail, even without the implementer changing anything in their code.

Ensuring this responsibility cannot be done by the compiler, unfortunately.  It thus becomes a developer's responsibility to ensure that any inheritance with method overriding does not introduce bugs to existing code.  This brings us to the _Liskov Substitution Principle_ (LSP), which says:

!!! note "Liskov Substitution Principle"
    "Let $\phi(x)$ be a property provable about objects $x$ of type $T$. Then $\phi(y)$ should be true for objects $y$ of type $S$ where $S <: T$."
    
    Barbara Liskov
    
    ---
    
    In English,
    
    > Is $S$ is a subclass of $T$, then an object of type $T$ can be replaced by that of type $S$ without changing the _desirable property_ of the program.
    
    or in simpler terms,
    
    > A subclass should not break the expectations set by the superclass.

This is consistent with the definition of subtyping, $S <: T$, but spelled out more formally.  We can use many ways to specify the _desirable property_ but the most common is the use of program specification.

!!! example "Example #1: Module"
    __Specification__
    
    We define a class called `Module` with a method `Module::marksToGrade`.  The desirable property of this method is that it takes in a `double` as input and returns a character (_i.e., of type_ `char`).  The character that are returned is one of the following: `'A'`, `'B'`, `'C'`, or `'F'`.  How the method `Module::marksToGrade` is implemented is irrelevant as long as it returns one of the characters above.
    
    __Usage__
    
    ```java
    void displayGrade(Module m, double marks) {
      char grade = m.marksToGrade(marks);
      if (grade == 'A')) {
        System.out.println("well done");
      } else if (grade == 'B') {
        System.out.println("good");
      } else if (grade == 'C') {
        System.out.println("ok");
      } else {
        System.out.println("retake again");
      }
    }
    ```
    
    __Subclass__
    
    Consider a subclass of `Module` called `CSCUModule`.  `CSCUModule` overrides `marksToGrade` method such that it now returns only `'S'` or `'U'`.  Does this violates LSP?
    
    Since `CSCUModule` is a subclass of `Module`, we can pass an instance to `displayGrade`:
    
    ```java
    displayGrade(new CSCUModule("GEQ1000", 100));
    ```
    
    and suddenly `displayGrade` is displaying `retake again` even if the student is scoring 100 marks.

The example above shows that we may be violating the LSP unintentionally.  The object `m` has the following property: `m.marksToGrade` always returns something from the set { `'A'`, `'B'`, `'C'`, `'F'` }, that the method `displayGrade` depends on explicitly.  The subclass `CSCUModule` violated that and makes `m.marksToGrade` returns `'S'` or `'U'`, sabotaging `displayGrade` and causing it to fail.

LSP cannot be enforced by the compiler[^1]. The properties of an object have to be managed and agreed upon among programmers.  A common way is to document these properties as part of the code documentation.

[^1]: We can use `assert` to check some of the properties though.

## LSP Through the Lens of Testing

Another way to develop an intuition of the LSP is through the lens of testing. When we write a method, we may want to introduce test cases to check that our method is working correctly. These test cases are designed based on the _specification_ of our method and not its implementation details[^2]. That is, we test based on the expected inputs and resultant outputs.

[^2]: The test cases we are describing here are known as black-box tests and you will encounter these in later modules at NUS. We will not go into any further details in this module.

!!! example "Example #2: Restaurant"
    __Specification__
    
    We would like to model a restaurant booking system for a restaurant chain. Every restaurant in the chain opens at 12 pm and closes at 10 pm, and has a singular method `canMakeReservation` which allows us to check if the restaurant is available for reservations at a certain `time`.  The requirement given is that, the system must be able to process a reservation during its opening hours.  On the other hand, we do not care about the behavior outside of the opening hour.
    
    We may then arrive at the following class:

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

    __Usage (_i.e., Test_)__
    
    The method `canMakeReservation` returns `true` when the argument passed in to `time` is between 12 pm and 10 pm. Let's think about how we would test this method.  Two important edge cases to test is to check if the method returns true for the stated restaurant opening and closing hours.

    ```Java
    Restaurant r = new Restaurant();
    r.canMakeReservation(1200) == true; // Is true, therefore test passes
      : // test for other hours between 1200 - 2200
    r.canMakeReservation(2200) == true; // Is true, therefore test passes
    ``` 

    Note that these are simple `jshell` tests, in software engineering modules (_e.g., CS2103/T_) you will learn better ways to design and formalise these tests.
    
    __Subclass #1__
    
    Consider a subclass of `Restaurant` called `LunchRestaurant` with the following implementation:

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
    
    `LunchRestaurant` does not take reservation during peak hours (_i.e., 1200 to 1400_).  As `LunchRestaurant` $<:$ `Restaurant`, we can point our variable `r` to a new instance of `LunchRestaurant` and run the test cases of the parent class, as can be seen in the code below.

    ```Java
    Restaurant r = new LunchRestaurant();
    r.canMakeReservation(1200) == true; // Is false, therefore test fails
      : // test for other hours between 1200 - 2200
    r.canMakeReservation(2200) == true; // Is true, therefore test passes
    ``` 
    
    Whilst the second test passes, the first test does not since it falls within the peak lunch hour.  Therefore `LunchRestaurant` is not substitutable for `Restaurant` and the LSP is violated.  We have changed the expectation of the method in the child class.
    
    __Subclass #2__
    
    Let's suppose the restaurant chain starts to roll out online reservation system for a subset of its restaurants.  These restaurants can take reservations any time.  We create a subclass `DigitalReadyRestaurant`, as follows:

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

Sometimes, it is useful for a developer to explicitly prevent a class to be inherited.  Not allowing inheritance would make it much easier to argue for the correctness of programs, something that is important when it comes to writing secure programs.  Both the two java classes you have seen, `java.lang.Math` and `java.lang.String`, cannot be inherited from.  In Java, we use the keyword `final` when declaring a class to tell Java that we ban this class from being inherited.

```Java
final class Circle {
    :
}
```

Alternatively, we can allow inheritance but still prevent a specific method from being overridden, by declaring a method as `final`.  Usually, we do this on methods that are critical for the correctness of the class.

For instance,
```Java
class Circle {
    :
  public final boolean contains(Point p) {
      :
  }
}
```

In short, we have learnt that the `final` keyword can be used in three places:

1. In a class declaration to prevent inheritance.
2. In a method declaration to prevent overriding.
3. In a field declaration to prevent re-assignment.

=== "Prevent Inheritance"
    __Code__
    
    ```java
    final class Circle {
        :
    }
    ```
    
    __Test__
    
    ```java
    class ColoredCircle extends Circle {
        :
    }
    ```
    
    __Error Message__
    
    ```
    _.java:_: error: cannot inherit from final Circle
    class ColoredCircle extends Circle {}
                                ^
    1 error
    ```
    
    __Note__
    
    Preventing inheritance also prevents overriding.
    
    ---

=== "Prevent Overriding"
    __Code__
    
    ```java
    class Circle {
        :
      public final boolean contains(Point p) {
          :
      }
    }
    ```
    
    __Test__
    
    ```java
    class ColoredCircle extends Circle {
        :
      @Override
      public final boolean contains(Point p) {
          :
      }
    }
    ```
    
    __Error Message__
    
    ```
    _.java:_: error: contains(Point) in ColoredCircle cannot override contains(Point) in Circle
    	public boolean contains(Point p) {
	                   ^
      overridden method is final
    1 error
    ```
    
    ---

=== "Prevent Re-Assignment"
    __Code & Test__
    
    ```java
    class Circle { 
      private Point c;
      private final double r;
      
      public Circle(Point c, double r) {
        this.c = c;
        this.r = r; // this is ok
      }
      
      public void setR(double r) {
        this.r = r; // this is re-assignment
      }
    }
    ```
        
    __Error Message__
    
    ```
    _.java:_: error: cannot assign a value to final variable r
            this.r = r;
                ^
    1 error
    ```
    
    __Note__
    
    The check if an assignment is the first assignment or a re-assignment may produce false positive.  In general, if the assignment to the field is done in the constructor, then it is allowed ___unless___ the field has already been initialized before (_e.g.,_ `private final double r = 0.0;`).  This is because a constructor can only be called once for each instance.
    
    On the other hand, if we have an assignment to a field not on the constructor like in the mutator, we cannot guarantee such a mutator will only be invoked exactly once.  Therefore, there is a possibility that there is a re-assignment.  The choice is to either prevent compilation or give a runtime error during the re-assignment.  Since compiler is our friend, they want to tell us about the problem early.
    
    ---
    