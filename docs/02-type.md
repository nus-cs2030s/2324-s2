# Unit 2: Variable and Type

!!! abstract "Learning Objectives"

    After this unit, students should
    
    - appreciate the concept of variables as an abstraction
    - understand the concept of types and subtypes
    - contrast between statically typed language vs. dynamically typed language
    - contrast between strongly typed language vs. weakly typed language
    - be familiar with Java variables and primitive types
    - understand widening type conversion in the context of variable assignments and how subtyping dictates whether the type conversion is allowed.

## Data Abstraction: Variable

One of the important abstractions that are provided by a programming language is the _variable_.  Data are stored in some location in computer memory.  But we should not be referring to the memory location all the time.  First, referring to something like `0xFA49130E` is not user-friendly; Second, the location may change.  A _variable_ is an abstraction that allows us to give a user-friendly name to a piece of data in memory.  We use the _variable name_ whenever we want to access the _value_ in that location, and a _pointer to the_ variable_ or _reference to the variable_ whenever we want to refer to the address of the location.

![Variable](figures/Variable.png)

## Type

As programs get more complex, the number of variables that the programmer needs to keep track of increases.  These variables might be an abstraction over different types of data: some variables might refer to a number, some to a string, some to a list of numbers, etc.  Not all operations are meaningful over all types of data.

To help mitigate the complexity,  we can assign a _type_ to a variable.  The type communicates to the readers what data type the variable is an abstraction over, and to the compiler/interpreter what operations are valid on this variable and how the operation behaves.  In lower-level programming languages like C, the type also informs the compiler how the bit representing the variable should be interpreted.

As an example of how types can affect how an operation behaves, let's consider
Python.  Suppose we have two variables `x` and `y`, storing the values `4` and `5` respectively and we run `print(x + y)`.

- If `x` and `y` are both strings, the output is `45`.
- If `x` and `y` are both integers, the output is `9`.
- If `x` is an integer and `y` is a string (or vice versa), you would get an error.

In the last instance above, you see that assigning a type to each variable helps to keep the program meaningful, as the operation `+` is not defined over an integer and a string in Python.

Java and Javascript, however,  would happily convert `4` into a string for you, and return `45`.

### Dynamic vs. Static Type

Python and Javascript are examples of _dynamically typed_ programming languages.  The same variable can hold values of different _unrelated_ types, and checking if the right type is used is done during the execution of the program.  Note that, the type is associated with the _values_, and the type of the variable changes depending on the value it holds.  For example, we can do the following:

=== "Javascript"

    ```Javascript
    let i = 4;   // i is an integer
    i = "5";     // ok, i is now a string
    ```

=== "Python"

    ```Python
    i = 4        // i is an integer
    i = "5"      // ok, i is now a string
    ```

Java, on the other hand, is a _statically typed_ language.  We need to _declare_ every variable we use in the program and specify its type.  A variable can only hold values of the same type as the type of the variable, so we can't assign, for instance, a string to a variable of type `int`.  Once a variable is _declared_ with a particular, the type of the variable cannot be changed.  In other words, the variable can only hold values of that declared type.

```Java
int i;   // declare a variable of type int
i = 4;   // ok because 4 is of type int
i = "5"; // error, cannot assign a string to an `int`
```

The type that a variable is assigned when we declare the variable is also known as the _compile-time type_.  During the compilation, this is the only type that the compiler is aware of.  The compiler will check if the compile-time type matches when it parses the variables, expressions, values, and function calls, and throw an error if there is a type mismatch.  This type-checking step helps to catch errors in the code early.

!!! important "Types on Variable"
    An important distinction between dynamic and static type is where the type gets **attached** to.  In static typing, the type is attached to the _variable_ such that the variable can only store values of that particular type (or its subtype as you will see later).  In fact, in Java, the type that is attached to a variable is the _declared_ type (_i,e._, the type written in the variable declaration also commonly known as compile-time type).

    On the other hand, in a dynamically typed language, the type is attached to the _value_.  In other words, a variable can store anything but we can know what the type is because the type can be queried from the value.

### Strong Typing vs. Weak Typing

A _type system_ of a programming language is a set of rules that governs how the types can interact with each other.  

A programming language can be strongly typed or weakly typed.  There are no formal definitions of "strong" vs. "weak" typing of a programming language, and there is a spectrum of "strength" between the typing discipline of a language.  

Generally, a _strongly typed_ programming language enforces strict rules in its type system, to ensure _type safety_, i.e., to ensure that if there are any problems with the program, it is not due to the type.  For instance, catching an attempt at multiplying two strings.  One way to ensure type safety is to catch type errors during compile time rather than leaving it to runtime.

On the other hand, a _weakly typed_ (or loosely typed) programming language is more permissive in terms of typing checking.  C is an example of a static, weakly typed language.  In C, the following is possible:

```Java
int i;        // declare a variable of type int
i = 4;        // ok because 4 is of type int
i = (int)"5"; // you want to treat a string as an int? ok, as you wish!   
```

The last line forces the C compiler to treat the string (to be more precise, the _address_ of the string) as an integer, through typecasting.

In contrast, if we try the following in Java:

```Java
int i;        // declare a variable of type int
i = 4;        // ok because 4 is of type int
i = (int)"5"; // error
```

we will get the following compile-time error message:

```
|  incompatible types: java.lang.String cannot be converted to int
```

because the compiler enforces a stricter rule and allows typecasting only if it makes sense.  More specifically, we will get a compilation error if the compiler can determine with **certainty** that such conversion can never happen successfully.

## Type Checking with A Compiler

In addition to checking for syntax errors, the compiler can check for type compilability according to the compile-time type, to catch possible errors as early as possible.  Such type-checking is made possible with static typing.  Consider the following Python program:

```Python
i = 0
while (i < 10):
  # do something that takes a long time
  i = i + 1
print("i is " + i)
```

Since Python does not allow adding a string to an integer, there is a type mismatch error on Line 5.  The type mismatch error is only caught when Line 5 is executed after the program is run for a long time.  Since the type of the variable `i` can change during run time, Python (and generally, dynamically typed languages) cannot tell if Line 5 will lead to an error until it is evaluated during run time.  

In contrast, statically typed language like Java can detect type mismatch during compile time since the compile-time type of a variable is fixed.  As you will see later, Java allows "addition" or string and integer, and but multiplication of a string and an integer.  If we have the following code, Java can confidently produce compilation errors without even running a program: 

```Java
int i = 0
while (i < 10) {
  // do something that takes a long time
  i = i + 1;
}
String s = "i is " * i;
```

## Primitive Types in Java

We now switch our focus to Java, particularly to the types supported.  There are two categories of types in Java, the _primitive types_ and the _reference types_.  We will first look at primitive types in this unit.

Primitive types are types that hold numeric values (integers, floating-point numbers) as well as boolean values (`true` _and_ `false`).  

For storing integral values, Java provides four types, `byte`, `short`, `int`, and `long`, for storing 8-bit, 16-bit, 32-bit, and 64-bit signed integers respectively.  The type `char` stores 16-bit unsigned integers representing UTF-16 Unicode characters.

For storing floating-point values, Java provides two types, `float` and `double`, for 32-bit and 64-bit floating-point numbers.

Unlike reference types, which we will see later, primitive type variables never share their value with each other, i.e., if we have:
```Java
int i = 1000;
int j = i;
i = i + 1;
```

`i` and `j` each store a copy of the value `1000` after Line 2.  Changing `i` on Line 3 does not change the content of `j`.

| Kinds | Types | Sizes |
|-------|-------|-------|
| Boolean | <ul><li>`boolean`</li></ul> | <ul><li>1-bit</li></ul> |
| Character | <ul><li>`char`</li></ul> | <ul><li>16-bit</li></ul> |
| Integral | <ul><li>`byte`</li><li>`short`</li><li>`int`</li><li>`long`</li></ul> | <ul><li>8-bit</li><li>16-bit</li><li>32-bit</li><li>64-bit</li></ul> |
| Floating-Point | <ul><li>`float`</li><li>`double`</li></ul> | <ul><li>32-bit</li><li>64-bit</li></ul> |

!!! info "Long and Float Constant"
    To differentiate between a `long` and an `int` constant, you can use the suffix `L` to denote that the value is expected to be of `long` type.  This is important for large values beyond the range of `int`.  On the other hand, if the constant is a floating-point constant, by default it is treated as type `double`.  You need to add the suffix `f` to indicate that the value is to be treated as a `float` type.

## Subtypes

An important concept that we will visit repeatedly in CS2030/S is the concept of subtypes.

!!! note "Subtype"
    Let $S$ and $T$ be two types.  We say that $T$ is a _subtype_ of $S$ if _a piece of code written for variables of type $S$ can also safely be used on variables of type $T$_.  

We use the notation $T <: S$ or $S :> T$ to denote that $T$ is a subtype of $S$.  The subtyping relationship in general must satisfy two properties:

1. **Reflexive**: For any type $S$, we have $S <: S$ (_i.e._, $S$ is a subtype of itself).
2. **Transitive**: If $S <: T$ and $T <: U$, then $S <: U$.  In other words, if $S$ is a subtype of $T$ and $T$ is a subtype of $U$, then $S$ is a subtype of $U$.

Additionally, in Java, you will find that the subtyping relationship also satisfies _anti-symmetry_.  However, this is often omitted as it is enforced by design.

- **Anti-Symmetry**: If $S <: T$ and $T <: S$, then $S$ must be the same type as $T$.

We also use the term _supertype_ to denote the reversed relationship: if $T$ is a subtype of $S$, then $S$ is a supertype of $T$.

### Subtyping Between Java Primitive Types

Considering the range of values that the primitive types can take, Java defines the following subtyping relationship:

- `byte` <: `short` <: `int` <: `long` <: `float` <: `double`
- `char` <: `int`

Graphically, we can draw the subtyping relationship as an arrow from subtype to supertype.  In the case of Java primitive types, we can visualise the subtyping relationship as follows:

![Primitive Subtype](figures/Primitive-Subtype.png)

!!! info "Long $<:$ Float?"
    Why is `long` a subtype of `float`?  More specifically, `long` is 64-bit and `float` is only 32-bit.  Clearly, there are more values in `long` than in `float`!

    The resolution lies in the precision.  While it is indeed true that any (mathematical) integer can be converted into a (mathematical) real number by simply appending `.0` to the end, some `long` values cannot be represented _precisely_ as `float`.  Try the following and see what is printed.

    ```Java
    long l = 123456789L;
    float f = l;
    System.out.println(l);
    System.out.println(f);
    ``` 

Given this, let us go back to the definition of subtype above and try to apply the definition to the code below that is written for variables of type `double`.

=== "Written for Double"

    ```Java
    double x = 5.0;    // the type S is double
    // code below is written assuming S = double
    double y = x + x;
    ```

=== "Used for Integer"

    ```Java
    int x = 5;         // the type S is now int
    // code below is written assuming S = double
    double y = x + x;  // still works! 
    ```

=== "Common Mistakes"

    ```Java
    double x = 5.0;    // the type S is double
    // code below is written assuming S = int
    int y = x + x;     // oh no! it does not work!
    ```

Valid subtype relationship is part of what the Java compiler checks for when it compiles.  Consider the following example:

=== "Code"

    ```Java
    double d = 5.0;
    int i = 5;
    d = i;
    i = d; // error
    ```

=== "What Compiler Sees"

    ```Java
    double d = 5.0;  // d::double <- 5.0::double
    int i = 5;       // i::int    <- 5::int
    d = i;           // d::double <- i::int (ok because int <: double)
    i = d;           // i::int    <- d::double (error because double </: int)
    ```

Line 4 above would lead to an error:

```
|  incompatible types: possible lossy conversion from double to int
```

but Line 3 is OK.  If you are still wondering why it does not work, you can click on the tab to see what the compiler "sees" in terms of compile-time type.

As you can see, the compile-time type of the variable `d` is `double` because that is what we declared it as.  Similarly, the compile-time type of the variable `i` is `int`.  Since `double` is the supertype of `int`, it can also store `int`.  On the other hand, an `int` cannot store `double`.

This example shows how subtyping applies to type checking.  _Java allows a variable of type $T$ to hold a value from a variable of type $S$ only if $S <: T$_.  This step is called _widening type conversion_.  Such conversion can happen during assignment or parameter passing.

!!! info "Why is it Called Widening?"
    The term widening is easy to see for primitive types with the exception of `long` to `float`.  You can immediately see that the subtype has fewer number of bits than the supertype.  Therefore, the supertype can have more possible values.  Hence, the size (_i.e._, the number of possible values) is wider.

    The opposite conversion is called _narrowing_ because the size is narrower.

## Additional Readings

- [Java Tutorial: Primitive Data Types](https://docs.oracle.com/javase/tutorial/java/nutsandbolts/datatypes.html) and other [Language Basics](https://docs.oracle.com/javase/tutorial/java/nutsandbolts/index.html)
