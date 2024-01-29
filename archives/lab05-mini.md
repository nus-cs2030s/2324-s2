# Lab 05 Mini Problem

## Learning Objectives

- Students can use nested class in their program.
- Students can organise their code into packages.

## Mini Problem 1
In this week's lecture, we've discussed about nested classes and see how it can be used. Now, let's try to write our own nested class. 

Instead of starting from scratch, let's go back to our `Company` class from Lab 1-3. 
For this mini problem, instead of having the subclasses as an outer class, let's combine them together and put everything inside `Company.java`.

To start, simply copy and paste all the class definition from `MNC.java`, `Startup.java`, `SME.java`, and `Gov.java` to the body of `Company.java`. 

> Question: How can you do this quickly using vim?

**Now that you have all your subclasses, what should these be declared as? _inner class_, _nested static class_, _local class_, or _anonymous class_? Can you use inner class and move `minResumeScore` and `minInterviewScore` to the parent class?**

Once you have figured out the answer, edit your main function to adhere to these changes.

## Mini Lab 2: Package and Protected
For the second part of the lab, you will learn about `package` and the `protected` modifier. This part is optional and only for your exploration.

### Packages
As the size of your program grows, it can be hard to locate and use the correct file to achieve a particular task. Just like how we usually organise things by putting them into groups (be it based on time, functionality, etc.), we can organise Java files into _packages_ which bundles related files together.

Starting from Lab 5 onwards, we will put our code inside a package instead of putting everything under `labX-username`.

To create a package, we choose a name for the package and put a package statement with that name at the top of every source file that we want to include in that package. 

Suppose we want to create a package called `shape`. To do so, you need to add the line:
```java
package shape;
```
on top of every `.java` file you want to include inside that package. In the `mini2` directory, you can see that `Circle.java`, `Square.java`, and `GetAreable.java` are all part of the `shape` package. To import these files, you can use:
```java
import shape.Circle;
```
as used in `TestOutside.java`.

### Protected
So far, we have been using two access modifiers: `private` and `public`. While the other modifiers won't be tested, just keep in mind that Java provides four access modifiers: `public`, `protected`, _no modifier_, and `private`. Their behavior are summarised by the table below:

| Modifier    | Class | Package | Subclass | World |
|-------------|-------|---------|----------|-------|
| public      | Y     | Y       | Y        | Y     |
| protected   | Y     | Y       | Y        | N     |
| no modifier | Y     | Y       | N        | N     |
| private     | Y     | N       | N        | N     |

You might also notice that we can set a class to be `public`. When it comes to (top level) class, you only have two option: `public` or _package-private_ (no modifier).  If a class has no modifier, then it is visible only within its own package.

To test your understanding, you can try to guess which lines in `TestShape.java` and `TestOutside.java` will cause compilation error. After you are done, you can play around with the codes under the `mini2` directory to see the behaviour of each access modifier.

```bash
.
├── shape
│   ├── Circle.java
│   ├── GetAreable.java
│   ├── Square.java
│   └── TestShape.java
└── TestOutside.java
```

`Circle.java`:
```java
package shape;

public class Circle {
  static int a = 0; 
  private static int b = 0; 
  protected static int c = 1; 
  public static int d = 0; 
}
```

`Square.java`:
```java
package shape;

class Square { // the class is not public
  static int w = 0; 
  private static int x = 0; 
  protected static int y = 1; 
  public static int z = 0; 
}
```

`TestShape.java`:
```java
package shape; // same package as Circle and Square

class TestShape {
  public static void main(String[] args) {
    System.out.println(Circle.a);
    System.out.println(Circle.b); 
    System.out.println(Circle.d);
    System.out.println(Circle.c);

    System.out.println(Square.w);
    System.out.println(Square.x); 
    System.out.println(Square.y);
    System.out.println(Square.z);
  }
}
```

`TestOutside.java`:
```java
// Different package (unnamed)
import shape.Circle;
import shape.Square;

class SubCircle extends Circle {
  public static int c = Circle.c; 
}

class TestOutside {
  public static void main(String[] args) {
    System.out.println(Circle.a); 
    System.out.println(Circle.b); 
    System.out.println(Circle.c); 
    System.out.println(Circle.d);

    System.out.println(Square.w); 
    System.out.println(Square.x); 
    System.out.println(Square.y); 
    System.out.println(Square.z);

    System.out.println(SubCircle.c);
  }
} 
```