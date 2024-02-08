# Exercise 2: Simulation 2

- Deadline: 14 February 2023, Wednesday, 23:59 SGT
- Difficulty Level: 5

## Prerequisite:

- Completed Exercise 1
- Caught up to Unit 19 of Lecture Notes
- Familiar with the CS2030S Java style guide

## Goal

This is a continuation of Exercise 1.  Exercise 2 changes some of the requirements of Exercise 1 and adds some new things to the world that we are simulating.  The goal is to demonstrate that, when OO principles are applied properly, we can adapt our code to changes in the requirement with less effort.  If your design for Exercise 1 follows the OOP principles, then only about 50 lines of changes/additions are required.

For Exercise 2, you should (i) improve upon your design for Exercise 1 if needed, (ii) update `BankSimulation` and associated classes to simulate the extended scenarios, and (iii) update the input and output components of the classes to conform to the specification below.

Exercise 2 also nudges you towards following good coding practice by adhering to a published coding convention.

### Extension 1: Simulating a Bank with a Queue

Recall that, no waiting was allowed inside the bank we are simulating.  The bank is losing customers as a customer departs if all the counters are busy.

Exercise 2 adds an entrance queue to the bank.  If all counters are busy when a customer arrives, the customer will join the queue and wait.  When a counter becomes available, the customer at the front of the queue will proceed to the counter for service.

The entrance queue has a maximum queue length of $m$.  If there are already $m$ customers waiting in the entrance queue, any arriving customer will be turned away[^1].

[^1]: This is known as "balking" in queueing theory.

If some counters are available when a customer arrives, the customer will go the first available counter, just like in Exercise 1.

### Extension 2: Customer with a Task

Customers now come to the bank with a task they intend to do.  The task can be either a deposit or a withdrawal. 
**Note:** We are not going to implement the withdrawal and deposit logic in our bank simulator yet.

### Changes to Input

1. There is an additional input parameter in the first line of the input file, an integer $m$, indicating the maximum allowed length of the entrance queue. This input parameter should be read immediately after reading the number of customers and the number of service counters.

2. There is an additional input parameter at the end of each line in the input file that describes a customer's arrival. The new parameter is an `int`, which is either $0$ (for deposit) or $1$ (for withdrawal).

### Changes to Output

1. A customer will now be printed with a single letter prefix `C`.  For instance, instead of `Customer 1`, we print `C1`.

2. A service counter will now be printed with a single letter prefix `S`.  For instance, instead of `Counter 1`, we print `S1`.

3. The entrance queue of the bank will be printed with the arrival event. E.g., the following shows that C3 arrived at time 1.400 and at the time of arrival, there were two customers, C1 and C2, waiting in the entrance queue.
```
1.400: C3 arrived [ C1 C2 ]
```

4. If a customer joins the entrance queue, the customer along with the queue _before_ joining should be printed. E.g.,
```
1.400: C3 joined queue [ C1 C2 ]
```

5. When a customer begins being served or is done being served, the task the customer is doing should be printed. For example, Customer `C2` withdrawing at bank counter `S1` would be printed as:
```
5.100: C2 withdrawal begin (by S0)
7.100: C2 withdrawal done (by S0)
```


## Skeleton for Exercise 2

We only provide two classes for Exercise 2, the main `Ex2.java` (which is simply `Ex1.java` renamed) and `Queue.java`.
The `Ex2.java` is similar to `Ex1.java`.  `Queue.java` is new.  It models a FIFO queue of objects.  Its usage will be explained further below.

Both files should not be modified for this exercise.

### Building on Exercise 1

You are required to build on top of your Exercise 1 submission for this exercise.

Assuming you have `ex1-<username>` and `ex2-<username>` under the same directory, and `ex2-<username>` is your current working directory, you can run
```
cp -i ../ex1-<username>/*.java .
rm -i Ex1.java
```

to copy all your Java code over and remove the main file for Ex1.

If you are still unfamiliar with Unix commands to navigate the file system and manage files, please review [our Unix guide](https://nus-cs2030s.github.io/2324-s2/unix/essentials.html).

You are encouraged to consider your tutor's feedback and fix any issues with your design for your Exercise 1 submission before you embark on your Exercise 2.

### The `Queue` class

`Queue` is a general class for a first-in, first-out queue of objects.  Here is an example of how it is used:

```Java
// Create a queue that holds up to 4 elements
Queue q = new Queue(4);

// Add a string into the queue.  returns true if successful; 
// false otherwise.
boolean b = q.enq("a1");

// Remove a string from the queue.  `Queue::deq` returns an 
// `Object`, so narrowing type conversion is needed.  Returns 
// `null` if queue is empty.
String s = (String) q.deq();

// Returns the string representation of the queue (showing 
// each element)
String s = q.toString();

// Returns true if the queue is full, false otherwise.
boolean b = q.isFull();

// Returns true if the queue is empty, false otherwise.
boolean b = q.isEmpty();

// Returns the number of objects in the queue
int l = q.length();
```

## Following CS2030S Style Guide

In addition to the changes above, you should also make sure that your code follows the [given Java style guide](https://nus-cs2030s.github.io/2324-s2/style.html)

You can use `checkstyle` tool and the given configuration `ex2_style.xml` to check your code:
```
java -jar ~cs2030s/bin/checkstyle.jar -c ex2_style.xml *.java
```

## Assumptions

We assume that no two events involving two different customers ever occur at the same time (except when a customer departs and another customer begins its service). As per all exercises, we assume that the input is correctly formatted.

## Compiling, Testing, and Debugging

### Compilation

To compile your code,
```Shell
javac *.java
```

### Running and Testing
You should not test your code by manually entering the inputs.  Instead, enter the inputs into a file, and run
```Shell
java Ex2 < file
```

A set of test inputs is provided as part of the skeleton, named `Ex2.x.in` under the `inputs` directory.  You can run them with, for instance,
```Shell
java Ex2 < inputs/Ex2.1.in
```

You can save the output by redirecting it into a file.
```Shell
java Ex2 < inputs/Ex2.1.in > OUT
```

You can automatically test your code against all the given inputs/outputs as well as against the `checkstyle` by running:
```Shell
./test.sh Ex2
```

### Debugging

The expected outputs are given in the `outputs` directory. You can compare `OUT` with the expected output with `diff` or `vim`.  Using `vim`,
```Shell
vim -d OUT output/Ex2.1.out
```

will open both files and highlight the differences.

As the output becomes too long, you can focus on tracing a particular counter or customer with the help of `grep`. Suppose you want to focus on what happened to Customer 1 in `OUT`, run
```Shell
grep ": C1" OUT
```

You should see the following output:
```
1.200: C1 arrived [ ]
1.200: C1 service begin (by S1)
2.200: C1 service done (by S1)
2.200: C1 departed
```

Suppose you want to see all the customers served by `S1`, run:
```Shell
grep "S1" OUT
```

You should see the following output:
```
1.200: C1 service begin (by S1)
2.200: C1 service done (by S1)
2.200: C4 service begin (by S1)
3.200: C4 service done (by S1)
```
