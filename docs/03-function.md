# Unit 3: Functions

!!! abstract "Learning Objectives"

    After this unit, students should

    - understand the importance of function as a programming constructor and how it helps to reduce complexity and mitigate bugs.
    - be aware of two different roles a programmer can play: the implementer and the client.
    - understand the concept of abstraction barrier as a wall between the client and the implementer, including in the context of a function.

## Function as an Abstraction over Computation

Another important abstraction provided by a programming language is the _function_ (or procedure).  This abstraction allows programmers to group a set of instructions and give it a name.  The named set of instructions may take one or more variables as input parameters, and return zero or one values.

Like all other abstractions, defining functions allows us to think at a higher conceptual level.  By composing functions at increasingly higher levels of abstraction, we can build programs with increasing levels of complexity.

Functions help us deal with complexity in a few ways as you will see later.  For now, to understand the examples, we will first describe a basic element of a function.  Note that this cannot be compiled but needs to be run on Jshell.  Once we have introduced the concept of a class, our code can be compiled.

!!! note "Syntax"
    The basic syntax of a function is as follows:

    ```Java
    return_type function_name(param_type1 param1, param_type2 param2) {
        : // function body
    }
    ```

    To create a function, we have to specify the return type.  If there is no return type, then we need to use the type called `void`[^1].  This is then followed by the function name.  We may have zero or more parameters for a function.

[^1]: `void` in Java is like a true nothingness (unlike Python's `None` or JavaScript's `undefined`).  If a function is declared as returning a type `void`, it cannot even be used in an assignment!

* Functions allow programmers to compartmentalize computation and its effects.  We can isolate the complexity within its body: the intermediate variables exist only as local variables that have no effect outside of the function.  A function only interacts with the rest of the code through its parameters and return value, and so, reduces the dependencies between variables to these well-defined interactions.  Such compartmentalization reduces the complexity of code.

    !!! example
        ```Java
        int factorial(int n) {
          if (n == 0) {
            return 1;
          } else {
            return n * factorial(n - 1);
          }
        }
        double e(int n) { // approximate e^n using Taylor series
          double x = 1;
          double res = 0;
          for (int i = 0; i < 10; i++) {
            res = res + (x/factorial(i));
            x = x * n;
          }
          return res;
        }
        // The n in factorial is different from the n in e.
        ```

* Functions allow programmers to hide _how_ a task is performed.  The caller of the function only needs to worry about _what_ the function does.  By hiding the details of _how_, we gain two weapons against code complexity.  First, we reduce the amount of information that we need to communicate among programmers.  A fellow programmer only needs to read the documentation to understand what the parameters are for, and what the return values are.  There is no need for a fellow programmer to know about the intermediate variables or the internal computation used to implement the functions.  Second, as the design and requirements evolve, the implementation of a function may change.  But, as long as the parameters and the return value of a function remain the same, the caller of the function does not have to update the code accordingly.  Reducing the need to change as the software evolves reduces the chances of introducing bugs accordingly.

    !!! example
        ```Java
        double sinc(double x) {
          return Math.sin(x)/x;
        }
        ```

        We do not need to know how to implement `sin(x)`, only whether `x` is in degree or radian.

* Functions allow us to reduce repetition in our code through _code reuse_.  If we have the same computation that we need to perform repeatedly on different _values_, we can construct these computations as functions by replacing the values with parameters, and pass in the values as arguments to the function.  This approach reduces the amount of boiler-plate code and has two major benefits in reducing code complexity and bugs.  First, it makes the code more succinct, and therefore easier to read and understand.  Second, it reduces the number of places in our code that we need to modify as the software evolves, and therefore, decreases the chance of introducing new bugs.

    !!! example
        ```Java
        double distance(double x1, double y1, double x2, double y2) {
            : // implementation omitted
        }
        boolean isEquilateral(double x1, double y1,
            double x2, double y2, double x3, double y3) {
          return distance(x1, y1, x2, y2) == distance(x2, y2, x3, y3)
              && distance(x1, y1, x2, y2) == distance(x1, y1, x3, y3);
        }
        // Define distance once, use it 4 times!
        // If we make a mistake in the function distance, we just have to correct it in one place
        //   instead of 4 different places if we did not use the function.
        ```

!!! info "Returning More Than One Return Value?"
    To return more than one value, we need to use a data type that can store multiple values.  At that point, it becomes a question of whether eating rice is eating one meal or hundreds of meals each consisting of a single grain of rice.

## Abstraction Barrier

We can imagine an _abstraction barrier_ between the code that calls a function and the code that defines the function body.  Above the barrier, the concern is about using the function to perform a task, while below the barrier, the concern is about _how_ to perform the task.  

While many of you are used to writing a program solo, in practice, you rarely write a program with contributions from only a single person.  The abstraction barrier separates the role of the programmer into two: (i) an _implementer_, who provides the implementation of the function, and (ii) a _client_, who uses the function to perform the task.  Part of the aim of CS2030/S is to switch your mindset into thinking in terms of these two roles.  In fact, in CS2030/S, you will be both but may be restricted to just being either a client or an implementer on specific functionality.

The abstraction barrier thus enforces a _separation of concerns_ between the two roles.  The client does not have to care how the implementer implements the functionality.  Similarly, the implementer does not have to care how the client is using the functionality as long as the client is following the specification of the functionality.

The concept of abstraction barrier applies not only to a function but it can be applied to different levels of abstraction as well.  We will see how it is used for a higher level of abstraction in the next unit.
