# Unit 15: Method Invocation

!!! abstract "Learning Objectives"

    Students should

    - understand the two step process that Java uses to determine which method implementation will be executed when a method is invoked.
    - understand that Class Methods do not support dynamic binding.

## How does Dynamic Binding work?

We have seen that, with the power of dynamic binding and polymorphism, we can write succinct, future-proof code[^1].  Recall that example below, where the magic happens in Line 4.  The method invocation `curr.equals(obj)` will call the corresponding implementation of the `equals` method depending on the run-time type of `curr`.

[^1]: Unless the future requires us to add functionality to every single classes.

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

How does dynamic binding work?  To be more precise, when the method `equals` is invoked on the target `curr`, how does Java decide which method implementation is this invocation bound to?  While we have alluded to the fact that the run-time type of the target `curr` plays a role, this is not the entire story.  Recall that we may have multiple versions of `equals` due to overloading.  So, Java also needs to decide, among the overloaded `equals`, which version of `equals` this particular invocation is bound to.

This unit elaborates on Java's decision process to resolve which method implemented in which class should be executed when a method is invoked.  This process is a two-part process.  The first occurs during compilation[^2]; the second during run time.

[^2]: While it is useful to think of this as a two-part process, what happens during compilation is really not so different from type checking.  In particular, it is trying to _guarantee_ that if the compilation is successful, the method to be invoked will definitely be found at run-time and will be able to accept the parameters of the specified compile-time type.

### During Compile Time 

During compilation, Java determines the method descriptor of the method invoked, using the compile-time type of the target.  For example, in the line

```Java
curr.equals(obj)
```

above, the target `curr` has the compile-time type `Object`.

Let's generalize the compile-time type of the target to $C$.  To determine the method descriptor, the compiler searches for all methods that can be _correctly_ invoked on the given argument.  Note that by correctly invoked, we simply look at the method **signature** that can _accept_ the given argument.  In other words, the type of parameter is the supertype[^3] of the compile-time type of the argument.  This method signature may be declared in class $C$ or the parent of $C$, or the grand-parent of $C$, and so on until we reach the root `Object`.

[^3]: Reminder, from our definition of subtype and supertype, a given type $S$ is both a subtype and a supertype of itself.  In terms of notation, we have $S <: S$, which is the same as $S :> S$.

In the example above, we look at the class `Object`, and there is only one method called `equals`.  The method can be correctly invoked with one argument of type `Object`.

What if there are multiple methods that can correctly accept the argument?  In this case, we choose the _most specific_ one.

!!! note "More Specific"
    Intuitively, a method $M$ is more specific than method $N$ if the arguments to $M$ can be passed to $N$ without compilation error.

    This may seem abstract so let's make it more concrete.  We will consider $A <: B <: C$.  Consider two method signature below.

    - `foo(A)`
    - `foo(C)`

    Now we consider the _"arguments to the method"_:

    - Arguments to `foo(A)`: type `A` and the subtype of `A`.
    - Arguments to `foo(C)`: type `A`, type `B`, type `C`, and the subtype of `A`.

    Note that the argument to `foo(A)` (_i.e., type_ `A` _and the subtype of_ `A`) can be passed to `foo(C)` without any compilation error because `foo(C)` can accept both type `A` and the subtype of `A`.

    On the other hand, the argument to `foo(C)` cannot be passed to `foo(A)`.  Take, for instance, type `B`.  It cannot be passed to `foo(A)`.

    So, `foo(A)` is more specific than `foo(C)`.

For example, let's say a class `Circle` implements:

```Java
boolean equals(Circle c) { .. }

@Override
boolean equals(Object c) { .. }
```

Then, `equals(Circle)` is more specific than `equals(Object)`.  Every `Circle` is an `Object`, but not every `Object` is a `Circle`.  Let's try to understand this in more details using the definition of "more specific" above.

First, in the phrase _"if the arguments to $M$ can be passed to $N$ without compilation error"_, we need to find what arguments can be accepted by the methods we wish to compare.  In the case of `equals(Circle)`, it can accept `Circle` (_and all its subclasses_).  On the other hand, for `equals(Object)`, it can accept `Object` and also `Circle`.

Now we simply have to test if `equals(Circle)` can accept whatever can be accepted by `equals(Object)` and vice versa.  So we test `equals(Circle)` and attempt to pass `Object` and `Circle` as argument (_since_ `equals(Object)` _can accept both_ `Object` _and_ `Circle`).  It will pass on `Circle` but it will fail on `Object`.

Similarly, we test `equals(Object)` and attempt to pass `Circle` (_since_ `equals(Circle)` _can only accept_ `Circle`).  There is definitely no compilation error here.  So now we have seen that if we set $M$ as the method `equals(Circle)` and $N$ as `equals(Object)`,

> the arguments to $M$ (_i.e.,_ `equals(Circle)`, _the argument is_ `Circle`) can be passed to $N$ (_i.e.,_ `equals(Object)` _can accept_ `Circle`) without compilation error.

Therefore, `equals(Circle)` is more specific than `equals(Object)`.  There is a possibility that comparing only two arbitrary methods, we will find that none of the two methods is more specific!  For instance, given `S1` <: `T` and `S2` <: `T`, `foo(S1)` is not more specific than `foo(S2)` and `foo(S2)` is not more specific than `foo(S1)`.

Once the method is determined, the method's descriptor (_return type and signature_) is stored in the generated code.
 
In the example above, the method descriptor `boolean equals(Object)` will be stored in the generated binaries.  Note that it does not include information about the class that implements this method.  The class to take this method implementation from will be determined in Step 2 during run-time.

### During Run Time

During execution, when a method is invoked, the method descriptor from Step 1 is first retrieved.  Then, the run-time type of the target is determined.

Let the run-time type of the target be $R$.  Java then looks for an _accessible_ (_i.e., not_ `private`) method with the matching descriptor in $R$.  If no such method is found, the search will continue up the class hierarchy, first to the parent class of $R$, then to the grand-parent class of $R$, and so on, until we reach the root `Object`.  The first method implementation with a matching method descriptor found will be the one executed.

For example, let's consider again the invocation in the highlighted line below again:

```Java hl_lines="3"
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

Let's say that `curr` points to a `Circle` object during run-time.  Suppose that the `Circle` class does not override the method `equals` in `Object`.  As a result, Java can't find a matching method descriptor `boolean equals(Object)` in the method `Circle`.  It then looks for the method in the parent of `Circle`, which is the class `Object`.  It finds the method `Object::equals(Object)` with a matching descriptor.  Thus, the method `Object::equals(Object)` is executed.

Now, suppose that `Circle` overrides the method `Object::equals(Object)` with its own `Circle::equals(Object)` method.  Since Java starts searching from the class `Circle`, it finds the method `Circle::equals(Object)` that matches the descriptor.  In this case, `curr.target(obj)` will invoke the method `Circle::equals(Object)` instead.

!!! info "Steps"
    The information above may be difficult to parse and understand.  So let's try to distill its essence into actionable steps.  In this example, we want to figure out the method invoked on `obj.foo(arg)`.

    #### Compile-Time Step

    1. Determine the compile-time type of `obj` (_i.e.,_ `CTT(obj)`).
    2. Determine the compile-time type of `arg` (_i.e.,_ `CTT(arg)`).
    3. Determine all the methods with the name `foo` that are _accessible_ in `CTT(obj)`.
        - This includes the parent of `CTT(obj)`, grand-parent of `CTT(obj)`, and so on.
    4. Determine all the methods from Step 3 that can accept `CTT(arg)`.
        - Correct number of parameters.
        - Correct parameter types (_i.e., supertype of_ `CTT(arg)`).
    5. Determine the _most specific_ method from Step 4.
        - If there is **no** most specific method, fail with compilation error.
        - Otherwise, record the method descriptor.

    #### Run-Time Step

    1. Retrieve the method descriptor obtained from _compile-time step_.
    2. Determine the run-time type of `obj` (_i.e.,_ `RTT(obj)`).
    3. Starting from `RTT(obj)`, find the first method that match the method descriptor as retrieved from Step 1.
        - If not found, check in the parent of `RTT(obj)`.
        - If not found, check in the grand-parent of `RTT(obj)`.
        - :
        - If not found, check in the root `Object`.
        - If not found, run-time error (_only happen on weird cases that we will not discuss as it involves shenanigans_).

    To see the steps in action, please follow the examples below.

!!! example
    Although the steps above are actionable, it is still instructive to at least see how the steps are carried out.  We will be using the following classes in our example.

    ```Java
    class U {
      void foo(T t) { }
      void foo(U u1, U u2) { }
    }

    class T extends U {
      void foo(S s) { }
    }

    class S extends T {
      void foo(U u) { }
    }
    ```

    Consider the following variables.

    ```Java
    U u = new T();
    S s = new S();
    ```

    In the last example, we will also show a tabular method that simplifies the steps.

    === "u.foo(s)"
        #### Compile-Time Step

        1. `CTT(obj)` = `U`
        2. `CTT(arg)` = `S`
        3. `foo` includes `foo(T)`, `foo(U, U)`
        4. `foo(S)` can be accepted by `foo(T)`
        5. The most specific is `void foo(T)`

        #### Run-Time Step

        1. Descriptor = `void foo(T)`
        2. `RTT(arg)` = `T`
        3. Check method from `T`
            - `void T::foo(T)` :material-close:
            - `void U::foo(T)` :material-check:

        ---

    === "u.foo(u)"
        #### Compile-Time Step

        1. `CTT(obj)` = `U`
        2. `CTT(arg)` = `U`
        3. `foo` includes `foo(T)`, `foo(U, U)`
        4. `foo(U)` cannot be accepted by any from Step 3.

        **Compilation-Error**

        ---

    === "s.foo(s)"
        #### Compile-Time Step

        1. `CTT(obj)` = `S`
        2. `CTT(arg)` = `S`
        3. `foo` includes `foo(U)`, `foo(S)`, `foo(T)`, `foo(U, U)`
        4. `foo(S)` can be accepted by `foo(U)`, `foo(S)`, `foo(T)`
        5. The most specific is `void foo(S)`

        #### Run-Time Step

        1. Descriptor = `void foo(S)`
        2. `RTT(arg)` = `S`
        3. Check method from `S`
            - `void S::foo(S)` :material-close:
            - `void T::foo(S)` :material-check:

    === "s.foo(s) Tabular"

        #### Compile-Time Step

        | `foo` | Accept `foo(S)` | Most specific |
        |-------|-----------------|---------------|
        | `foo(U)` | :material-check: | :material-close: |
        | `foo(S)` | :material-check: | :material-check: |
        | `foo(T)` | :material-check: | :material-close: |
        | `foo(U, U)` | :material-close: | - (_can be ignored_) |

        #### Run-Time Step

        | Class | Has `void foo(S)`? |
        |-------|--------------------|
        | `S` | :material-close: |
        | `T` | :material-check: |
        | `U` | - (_can be ignored_) |

        ---

    Hopefully the 3 examples above are informative.  We cannot cover all possibilities but the steps should provide guidance on what to do when new situation arise.  In recitation we will connect this with class diagram so that we can understand the steps visually.

## Invocation of Class Methods

The description above applies to instance methods.  Class methods, on the other hand, do not support dynamic binding.  The method to invoke is resolved statically during compile time.  The same process in Step 1 is taken, but the corresponding method implementation in class $C$ will always be executed during run-time, without considering the run-time type of the target.

!!! danger "Bad Practice"
    To show that invocation of class methods is not via dynamic binding, we have to use example that are basically frowned upon.  Do not follow this bad practice in your coding.

    ```Java
    class T {
      static int f() {
        return 1;
      }
    }

    class S extends T {
      static int f() {
        return 2;
      }
    }
    ```

    Typically, we will invoke the static method `f` using either `T.f()` or `S.f()`.  Unfortunately, to show that invocation of class method is using static binding, we have to use the instance rather than the class.  Which leads us to the unorthodox code snippet below.

    ```Java
    T t = new S(); // compile-time type is T, run-time type is S
    System.out.println(t.f()); // what is the expected result?
    ```