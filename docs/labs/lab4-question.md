# Lab 4: Some Body Once Told Me

!!! abstract "Basic Information"
    - __Deadline:__ 11 October 2023, Tuesday, 23:59 SST
    - __Marks:__ 20
    - __Weightage:__ 2%

!!! info "Prerequisite"
    - Caught up to [Unit 27](../27-inference.md) of Online Notes.

!!! note "Files"
    In the directory, you should see the following files:

    1. __Java Files:__
        - `Some.java`: A template for `Some` class.
        - `JackInTheBox.java`: A template for `JackInTheBox` class.
        - `Transformer.java`: A template for `Transformer` interface.
        - `A.java`, `B.java`, `C.java`: Classes for flexibility check.
        - `CS2030STest.java`, `Test1.java` to `Test4.java`: Tester class

## Preliminary

In this lab, we are going to build our own generic wrapper class, a `Some<T>`.  This is a wrapper class that can be used to store an item of any reference type.  For this lab, our `Some<T>` is not going to be a very useful abstraction.  Not to worry. we will slowly add more functionalities to it later in this module.

In the following, we will slowly build up the `Some<T>` class along with some additional interfaces.  We suggest that you develop your class step-by-step in the order below.


## Task 1: A Simple Container

Build a generic class `Some<T>` that

- contains a `private` `final` field of type `T` to store the content.
- overrides the `boolean equals(Object)` method from `Object` to compare if two containers are the same.  Two containers are the same if the contents are equal to each other, as decided by their respective `equals` method.
- overrides the `String toString()` method so it returns the string representation of its content, between `[` and `]`.
- provides a class method called `some` that returns a container with a given object.  You may assume that no `null` value will ever be given.  This restriction will be lifted in lab 5.

!!! info "Factory Method"
    The method `some` is called a factory method. A factory method is a method provided by a class for the creation of an instance of the class.  Using a public constructor to create an instance necessitates calling `new` and allocating a new object on the heap every time.  A factory method, on the other hand, allows the flexibility of reusing the same instance.  The `some` method does not currently reuse instances but this will be rectified in lab 5.

With the availability of the of factory method, `Some<T>` should keep the constructor `private`.

### Sample Usage

The sequence below shows how we can use a `Some` using the methods you developed above.

```
jshell> Some.<Integer>some(4)
$.. ==> [4]
jshell> Some.some(5) // type inference!
$.. ==> [5]

jshell> Some.some(4).equals(Some.some(4))
$.. ==> true
jshell> Some.some(4).equals(4)
$.. ==> false
jshell> Some.some(Some.some(0)).equals(Some.some(Some.some(0)))
$.. ==> true
jshell> Some.some(Some.some(0)).equals(Some.some(0))
$.. ==> false
jshell> Some.some(0).equals(Some.some(Some.some(0)))
$.. ==> false

jshell> Some.some("body once told me")
$.. ==> [body once told me]
jshell> Some.some("4").equals(Some.some(4))
$.. ==> false
```

You can test your `Some<T>` more comprehensively by running:

```
javac -Xlint:rawtypes Test1.java
java Test1
```

There shouldn't be any compilation warning or error when you compile `Test1.java` and all tests should prints `ok`.


## Task 2: Transformation

Now, we are going to write an interface (_along with its implementations_) and a method in `Some` that allows a container to be transformed into another container, possibly containing a different type.

### Step 1: Transformer Interface

First, create an interface called `Transformer<T, U>` with an abstract method called `transform` that takes in an argument of generic type `T` and returns a value of generic type `U`.

### Step 2: Mapping Method

Second, write a method called `map` in the class `Some` that takes in a `Transformer`, and use the given `Transformer` to transform the container (_and the value inside_) into another container of type `Some<U>`.  You should leave the original container unchanged.

Make sure that you make the method signature as _flexible_ as possible.  Follow the PECS principle after you determine which type (_i.e.,_ `T` _or_ `U`) acts as producer or consumer (_or both?_).

### Sample Usage

```
jshell> class AddOne implements Transformer<Integer, Integer> {
   ...>   @Override
   ...>   public Integer transform(Integer arg) {
   ...>     return arg + 1;
   ...>   }
   ...> }
jshell> class StrLen implements Transformer<String, Integer> {
   ...>   @Override
   ...>   public Integer transform(String arg) {
   ...>     return arg.length();
   ...>   }
   ...> }
jshell> AddOne fn1 = new AddOne()
jshell> StrLen fn2 = new StrLen()

jshell> Some.some(4).<Integer>map(fn1)
$.. ==> [5]
jshell> Some.some(5).map(fn1)
$.. ==> [6]

jshell> Some<Number> six = Some.some(4).map(fn1).map(fn1)
six ==> [6]
jshell> six.map(fn2)
|  Error: ...
|  six.map(fn2)
|  ^-----^

jshell> Some<String> mod = Some.some("CS2030S")
mod ==> [CS2030S]
jshell> mod.map(fn2)
$.. ==> [7]
jshell> mod
mod ==> [CS2030S]
jshell> mod.map(fn2).map(fn1)
$.. ==> [8]
```

You can test your additions to `Some<T>` more comprehensively by running:

```
javac -Xlint:rawtypes Test2.java
java Test2
```

There shouldn't be any compilation warning or error when you compile `Test2.java` and all tests should prints `ok`.


### Flexible Usage

```
jshell> /open A.java
jshell> /open B.java
jshell> /open C.java

jshell> class AtoC implements Transformer<A, C> {
   ...>   @Override
   ...>   public C transform(A arg) {
   ...>     return new C(arg.get());
   ...>   }
   ...> }
jshell> class BtoB implements Transformer<B, B> {
   ...>   @Override
   ...>   public B transform(B arg) {
   ...>     return new B(arg.get());
   ...>   }
   ...> }
jshell> class CtoA implements Transformer<C, A> {
   ...>   @Override
   ...>   public A transform(C arg) {
   ...>     return new A(arg.get());
   ...>   }
   ...> }

jshell> Some<A> someA = Some.some(new A(1))
jshell> Some<B> someB = Some.some(new B(2))
jshell> Some<C> someC = Some.some(new C(3))
jshell> AtoC fn1 = new AtoC()
jshell> BtoB fn2 = new BtoB()
jshell> CtoA fn3 = new CtoA()

jshell> someA.map(fn1)
$.. ==> [C:1]
jshell> someA.map(fn2)
|  Error: ...
|  someA.map(fn2)
|  ^-------^
jshell> someA.map(fn3)
|  Error: ...
|  someA.map(fn3)
|  ^-------^

jshell> someB.map(fn1)
$.. ==> [C:2]
jshell> someB.map(fn2)
$.. ==> [B:2]
jshell> someB.map(fn3)
|  Error: ...
|  someB.map(fn3)
|  ^-------^

jshell> someC.map(fn1)
$.. ==> [C:3]
jshell> someC.map(fn2)
$.. ==> [B:3]
jshell> someC.map(fn3)
$.. ==> [A:3]
```

You can test the flexibility of `map` on `Some<T>` more comprehensively by running:

```
javac -Xlint:rawtypes Test3.java
java Test3
```

There shouldn't be any compilation warning or error when you compile `Test3.java` and all tests should prints `ok`.


## Task 3: Jack in the Box

The `Transformer` interface allows us to transform the content of the container from one type into any other type, including a `Some<T>`! You have seen examples above where we have a container inside a container: `Some.some(Some.some(0))`.

Now, implement your own `Transformer` in a class called `JackInTheBox<T>` to transform an item into a `Some` containing the item. The corresponding type `T` is transformed into `Some<T>`. This transformer, when invoked with `map`, results in a new `Some` within the `Some`.

### Sample Usage

```
jshell> Some.some(4).map(new JackInTheBox<>())
$.. ==> [[4]]
jshell> Some.some(Some.some(5)).map(new JackInTheBox<>())
$.. ==> [[[5]]]
```

You can test your `JackInTheBox<T>` more comprehensively by running:

```
javac -Xlint:rawtypes Test4.java
java Test4
```

There shouldn't be any compilation warning or error when you compile `Test4.java` and all tests should prints `ok`.

## Check Style

To run the style checker,

```
$ java -jar ~cs2030s/bin/checkstyle.jar -c ~cs2030s/bin/cs2030_checks.xml *.java
```

While we try to keep the files we give conform to our style guide, in the case any of the files we gave has style problems, you do not have to correct them.


## Grading

This lab is worth 20 marks and contribute 2% to your Lab Assignment component.  The marking scheme is as follows:

| Component | Sub-Component | Marks |
|-----------|---------------|-------|
| Correctness | | 16 marks |
| | _Test 1 related<br>Test 2 related<br>Test 3 related<br>Test 4 related_ | _4 marks<br>4 marks<br>4 marks<br>4 marks_ |
| Style | | 4 marks |

Correctness mark will only be awarded if your code compiles for the given test files, otherwise we cannot run any tester.  While we try to give you all the possible tests, there may be some tests we missed.  You should not hardcode any test cases and you should expect potentially additional test cases.

Additionally, if your code cannot compile __for any reason__, you will get __0%__ of the mark for styles.

We may make additional deductions for other issues or errors in your code such as run-time error, failure to follow instructions, not commenting `@SuppressWarning`, misuse of `@SuppressWarning` (_unnecessary, not in smallest scope, etc_), etc.

!!! info "Suppress Warnings"
    If you design your code correctly, you do not need any `@SuppressWarnings`.  If you have any, you may want to check your design again.

## Submission

To submit the lab, run the following command from the directory containing your lab 4 code.

```sh
~cs2030s/submit-lab4
```

Please check your repo after running the submission script to ensure that your files have been added correctly.  The URL to your repo is given after you run the submission script.

!!! danger "Do NOT Use Other Git Command"
    While you may be familiar with git commands, please do not use them.  Please use only the submission script `submit-labX` to ensure that your submissions are recorded properly.