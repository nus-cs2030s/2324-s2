# Unit 1: Program and Compiler


!!! abstract "Learning Objectives"

    Students should
    
    - recap some fundamental programming concepts, including the concept of a program, a programming language, a compiler, an interpreter.
    - be aware of two modes of running a Java program (_compiled vs. interpreted_).
    - be aware that compile-time errors are better than run-time errors, but the compiler cannot always detect errors during compile time.


## Software Program

A software program is a collection of data variables and instructions on how to modify these variables.  To dictate these instructions to the computer, programmers usually write down the instructions using a programming language, expressing their instructions in code that are made up of keywords, symbols, and names.  

A programming language is a formal language that helps programmers specify _precisely_ what the instructions are at a higher level of _abstraction_ (_i.e., at a higher conceptual level_) so that a programmer only needs to write a few lines of code to give complex instructions to the computer.  


## Compiled vs. Interpreted Programs

The processing unit of a computer can only accept and understand instructions written in machine code.  A program, written in a higher-level programming language, therefore needs to be translated into machine code before execution.  There are different approaches to how such translations can be done.  The first approach uses a _compiler_ -- a software tool that reads in the entire program written in a higher-level programming language and translates it into machine code.  The machine code is then saved into an executable file, which can be executed later.  `clang`, a C/C++ compiler, is an example.  The second approach uses an _interpreter_ -- software that reads in the program one statement at a time interprets what the statement means, and executes its directly.   This is how Python and Javascript programs are executed. 

Modern programming systems for executing programs are, however, more sophisticated.  V8, for instance, is an open-source engine that executes Javascript, and it contains both an interpreter that first interprets a Javascript into _bytecode_ (_an intermediate, low-level representation_).  A just-in-time compiler then reads in the bytecode and generates machine code dynamically at runtime with optimized performance. 

Java programs, on the other hand, can be executed in two ways:

1. The Java program can first be compiled into bytecode.  During execution, the bytecode is interpreted and compiled on-the-fly by the _Java Virtual Machine_ (_JVM_) into machine code.
2. The Java program can be interpreted by the Java interpreter.

To better understand this distinction, we will introduce a visual aid to describe the relationships between programs, compilers, interpreters, and machines.

### Tombstone Diagrams (_T-Diagrams_)
Tombstone Diagrams or T-diagrams consist of combinations of the following four components: 

- *Programs* which are implemented in a particular language (_i.e.,_ `Java`, `python`, `c/c++`)
- Language **A** *Interpreters* implemented in language **B**
- Language **A** to language **B** *Compilers* which are implemented in a language **C**
- Physical *Machines* implementing a particular language (_i.e., x86-64, ARM-64_)

These components are represented in T-diagrams as shown in the figure below:

![Compiler 001](figures/compiler/compiler.001.png){ width=500 }

We can treat these components like "puzzle pieces" and build diagrams to describe various execution, compilation, or interpreted processes.  For example, in the diagram below, a python script `Hello.py` is being interpreted by the python interpreter running on the x86-64 architecture.

![Compiler 002](figures/compiler/compiler.002.png){ width=375 }

*Note:* In order for the diagram to be valid, adjacent connected diagram components need to match. This can be seen in the diagram below (_highlighted with blue boxes_).

![Compiler 006](figures/compiler/compiler.006.png){ width=700 }

Since CS2030/S uses Java, we will now look at the two approaches to execute a Java program in more detail (_without worrying about how to write a single line of Java first_).

### Compiling and Running Java Programs

Suppose we have a Java program called `Hello.java`.  To compile the program, we type[^1]

```
$ javac Hello.java
```

into the command line.  `javac` is the Java compiler.  This step will either lead to the bytecode called `Hello.class` being created or spew out some errors.  This process can be seen in the figure below, where the `Hello.java` program is compiled from Java to the JVM language (_bytecode_). The Java compiler `javac` in this diagram is implemented in the x86-64 machine language.

![Compiler 003](figures/compiler/compiler.003.png){ width=700 }

Assuming that there is no error in compilation, we can now run

```
$ java Hello
```

to invoke the JVM `java` and execute the bytecode contains in `Hello.class`. This can be seen in the figure below, where the `Hello.class` program is interpreted from JVM language (_bytecode_) to the x86-64 machine language.

![Compiler 004](figures/compiler/compiler.004.png){ width=375 }

[^1]: The `$` represents the command prompt in a shell and you do not need to type this.

Beginners tend to confuse between `javac` and `java`, and whether to add the extension `.java` or `.class` when compiling and executing a Java program.  Do take note and refer back here if needed[^2].

[^2]: To add to the confusion, Java 11 introduces a shortcut where we can run `java Hello.java` directly.  This command causes `Hello.java` to be compiled and executed in a single step.  We won't do this in CS2030/S (i) to avoid confusion and (ii) to show you the steps explicitly.

!!! failure "Common Mistakes"
    A common mistake when compiling and running Java program is to use the `java` command on a `.java` file instead of the `javac` command.
    To make matter worse, you can **some time** run the `.java` program using the `java` command seemingly bypassing the need for compilation using `javac` command.
    This feature is called ["Launch Single-File Source-Code Programs"](https://openjdk.org/jeps/330).
    In general, avoid the following command.

    ```
    $ java Hello.java
    ```

### Interpreting a Java program

Java (_version 8 or later_) comes with an interpreter called `jshell` that can read in Java statements, evaluate them, and print the results[^3]. `jshell` is useful for learning and experimenting about Java.   This can be seen in the figure below, where the `Hello.java` program is interpreted from Java directly to the x86-64 machine language. 

![Compiler 005](figures/compiler/compiler.005.png){ width=375 }

To run `jshell` in interactive mode, we type

```
$ jshell
```

on the command line, which causes an interactive prompt to appear:

```
$ jshell
|  Welcome to JShell -- Version 17
|  For an introduction type: /help intro

jshell>
```

We can now type in Java code on `jshell>`. 

Alternatively, we can also include all the Java statements that we want `jshell` to run in a file and pass it into jshell

```
$ jshell Hello.jsh
```

[^3]: Such a program is called REPL (Read-Evaluate-Print in a Loop) for short.

While `jshell` is a convenient way to test things out and play with new Java concepts as we learn, do keep in mind that `jshell` combined both compilation and run-time into a single step.  The error spewed out by `jshell` could be either compile-time error or run-time error, and this could be confusing to beginners who try to distinguish between the two phases of program execution.

!!! note "jsh vs java"
    Files intended to be run on jshell typically uses `.jsh` extension while files intended to be compiled and run uses `.java` extension.  However, this difference is merely a convention.  You can still interpret `.java` program on jshell.

## Compiler

!!! quote
    _"One man's program is another program's data."_

    Olivier Danvy

The compiler does more than just translating source code into machine code or bytecode.  The compiler also needs to parse the source code written and check if it follows the precise specification of the programming language (_called grammar_) used, and produces a _syntax error_ if the grammar is violated.  It therefore can detect any syntax error before the program is run.

It is much better for a programmer to detect any error in its code during compilation -- since this is the phase when the program is still being developed and under the control of the programmer.  Runtime error, on the other hand, might occur when the customers are running the program, and so are much less desirable.  As such, we try to detect errors as much as possible during compilation.  The compiler is a powerful friend for any programmer if used properly.

The power of the compiler, however, is limited.  A compiler can only read and analyze the source code without actually running it.  Without running the program, the compiler cannot always tell if a particular statement in the source code will ever be executed; it cannot always tell what values a variable will take.

To deal with this, the compiler can either be _conservative_, and report an error as long as _there is a possibility_ that a particular statement is incorrect; or, the compiler can be more _permissive_, reporting an error only if _there is no possiblity_ that a particular statement is correct.  If there is a possibility that a particular statement is correct, it does not throw an error, but rely on the programmer to do the right thing.  We will further contrast these two behaviors later in this module.

## Workflow

A typical workflow in a compiled language is the edit, compile, execute, loop.  To start, we first either create or edit the file containing our Java program (_e.g.,_ `Hello.java`) with our favourite editor.  We recommend the use of [`vim`](vim/philosophy.md) editor.  This will be our source code.

We then compile our source code using a compiler (_e.g.,_ `javac Hello.java`) to produce a `.class` file.  If the program compiles, then we proceed with the next step. Otherwise, we go back to editing our source code.

The next step is to execute the program and test.  If we find incorrect result in our testing, we restart from the first step again.  This step is performed until we are satisfied with our program or until we have to submit our program.  The diagram below summarises the steps.

![Workflow](figures/Workflow.png){ width=700 }

!!! info "Steps"
    === "Edit"
        1. Create a new Java program file with the extension `.java` or edit an existing file.
            - We would recommend the use of [`vim`](vim/philosophy.md) as it is available on PE node so you can work almost anywhere with an active internet connection.
            - Using `vim`, you can type `vim Hello.java`.
        2. Save the file (_e.g.,_ `Hello.java`).
            - To save an exit on `vim`, the command is `:wq`.

    === "Compile"
        1. Compile your Java program from the "Edit" step using `javac` (_e.g.,_ `Hello.java`).
            - The command is `javac Hello.java`.
        2. You will either
            - produce a bytecode (_e.g.,_ `Hello.class`), then you continue to "Execute" step, _or_
            - fail to compile (_i.e., compilation error_), then you repeat the "Edit" step to repair the program and remove the source of the compilation error according to the given error message.

    === "Execute"
        1. Execute your bytecode (_e.g.,_ `Hello.class`) using `java`.
            - The command is `java Hello`.
        2. You will either
            - get the _correct_ program output and you are satisfied with your work and finish, _or_
            - get the _incorrect_ program output, then your repeat the "Edit" and "Compi;e" step to repair your program until you get the correct program output or you run out of time.