# Lab 03 Mini Problems

## Learning Objectives

- Students can incorporate generic array structures into their code.

## Mini Problem 1

This problem is an extension to mini problem 1 of Lab 2.  The solution to the mini problem 1 of Lab 2 can be found in the accompanying zip file.

### Task 1: Creating Generic Stack

Currently, we used an `ArrayList` of companies.  But now that we have learnt about generics, we can make our own generic data structure instead of using the .  That data structure is a stack.

A stack is a _last in, first out_ data structure.  In other words, the last element you put in will be the first one taken out.  Similar to how from a stack of paper, the top is the last element you put in.  In the same way, the top paper is the first element taken out.

Implement a `Stack<T>` in the file `Stack.java` with the following condition:

- `Stack<T>` takes in only a subtype of `Company` as its type argument.
- `Stack<T>` has a constructor that takes in a single integer corresponding to the maximum size of the stack.
- `Stack<T>` supports the following methods:
    - `void push(T elem)`: insert the element `elem` of type `T` to the top of the stack.
        - Do nothing if the stack is full.
    - `T pop()`: remove and return the top element of the stack.
        - Return `null` if the stack is empty.
    - `boolean isEmpty()`: returns `true` if the stack is empty.

### Task 2: Using Generic Stack

Modify the class `Lab03Mini1` to remove any dependency to `ArrayList<Company>` and use `Stack<Company>` instead.



## Mini Problem 2

This problem is an extension to mini problem 2 of Lab 2.  The solution to the mini problem 2 of Lab 2 can be found in the accompanying zip file.


### Task 1: Creating Generic Queue

Currently, we used an `ArrayList` of employee.  But now that we have learnt about generics, we can make our own generic data structure instead of using the .  That data structure is a queue.

A queue is a _first in, first out_ data structure.  In other words, the first element you put in will be the first one taken out.  Similar to how from a queue at the canteen, you enter from the back of the queue.  Then, when you reached the front, you may exit the queue.

Implement a `Queue<T>` in the file `Queue.java` with the following condition:

- `Queue<T>` takes in only a subtype of `Employee` as its type argument.
- `Queue<T>` has a constructor that takes in a single integer corresponding to the maximum size of the queue.
- `Queue<T>` supports the following methods:
    - `void enqueue(T elem)`: insert the element `elem` of type `T` to the back of the queue.
        - Do nothing if the queue is full.
    - `T dequeue()`: remove and return the top element of the queue.
        - Return `null` if the queue is empty.
    - `boolean isEmpty()`: returns `true` if the queue is empty.

### Task 2: Using Generic Queue

Modify the class `Payroll` to remove any dependency to `ArrayList<Employee>` and use `Queue<Employee>` instead.