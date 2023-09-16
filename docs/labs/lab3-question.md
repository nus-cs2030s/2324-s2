# Lab 3: Priority

!!! abstract "Basic Information"
    - __Deadline:__ 19 September 2023, Tuesday, 23:59 SST
    - __Marks:__ 20
    - __Weightage:__ 2%

!!! info "Prerequisite"
    - Completed Lab 2.
    - Caught up to [Unit 25](../25-unchecked.md) of Online Notes.

!!! note "Files"
    In the directory, you should see the following files:

    1. __Java Files:__
        - `Packet.java`: A template for a packet.
        - `PacketTest.java`: A tester for packet.
        - `BufferTest.java`: A tester for buffer.
        - `Lab3.java`: The main program that is simply a renaming of `Lab2.java`.
    2. __Input/Output Files:__
        - `inputs/Lab3.k.in` for the input files.
        - `outputs/Lab3.k.out` for the output files.
    3. __Bash Script:__
        - `test.sh`: Testing `Lab3` by comparing the output when running `Lab3` on `inputs/Lab3.k.in` with the expected output in `outputs/Lab3.k.out`.

## Restriction

In this lab, you are not allowed to use any built-in libraries from Java related to sequences for your implementation of `Buffer<T>`.  This includes _but not limited to_ `List`, `ArrayList`, `PriorityQueue` (_except the one initially already given in_ `Network.java`), _etc_.  You are also not allowed to use `java.util.Arrays`.

## Preliminary

Currently, our network is missing two main components.  The first is that we did not encapsulate a packet but simply uses a string.  Secondly, our buffer does not allow for a priority message.

There will be no major changes to the sender and receiver in this lab.  Instead, we will focus our attention to the buffer.


## Task 0: Preparation

### Copy

Assuming you have `lab2-<username>` and `lab3-<username>` under the same directory.  And assuming `lab3-<username>` is your current working directory (_you can check with_ `pwd` _command_).  You can execute the following command to copy the `.java` files from your Lab 2 to Lab 3.

```bash
cp -i ../lab2-<username>/*.java .
```

### Starting

Similar to before, you are encouraged to consider your tutor's feedback and fix any issues with your design for your Lab 2 submission before you embark on your Lab 3 journey.

## Task 1: Packet

Your firt task is to create a packet class that implements `Comparable<Packet>`.  In other words, two packets can be ordered.  We order packets based on length as follows:

- If packet `P1` has shorter length that packet `P2`, then `P2` < `P1`.
- If packet `P1` has the same length as as packet `P2`, then `P1` = `P2`.
- If packet `P1` has longer length that packet `P2`, then `P2` > `P1`.

In other words, packet with shorter length has larger "order".  To connect this with priority, we want to prioritize shorter messages over longer messages.

`Packet` should satisfy the following:

- A packet should implement `Comparable<Packet>`.
- A packet should have a field of type `String`.
- The constructor of a packet should take in a single parameter `String`.
    - This constructor assigns the parameter to the field.
- A packet should have a `toString` method to print the message.
- A packet should have a `compareTo` method to compare according to the specification above.
- A packet should __NOT__ have any other methods.

### Testing

You can test your `Packet` class by running the following.  You are also encouraged to __add more tests__.

```
javac -Xlint:rawtypes -Xlint:unchecked PacketTest.java
java PacketTest
```

## Task 2: Buffer

We now need to make `Buffer` a generic class.  In other words, we want to make `Buffer<T>`.  You may build upon `Array<T>` from the lecture.

- `Buffer<T>` takes in only a subtype of `Comparable<T>` as its type arguments.
    - We parameterize `Buffer<T>` with only `T` that can be compared with itself.
    - You may use `@Suppresswarnings("unchecked")` __at the smallest scope possible__ to suppress warnings about unchecked cast __only if needed__.
- `Buffer::send` should now accept `T` instead of `String`.
    - The behavior of `Buffer::send` is still the same as in Lab 2.
- `Buffer::receive` should now return `T` instead of `String`.
    - Instead of removing and returning the element at index 0, you should find the element with the largest priority.
        - Recap that shorter messages have larger priority.
    - If there is more than one messages with the largest priority, we choose the one with the smallest index.
    - Remove and return the element with the largest priority.

Since you cannot use `PriorityQueue`, `ArrayList`, _etc_, it is useful to be able to "compactify" an array.  Since this is not CS2040/S, we will give you the code to do this.

Consider an `Integer` array `arr` (_i.e.,_ `Integer[] arr`).  If we wish to remove element at index `idx` and keep the array in the same order but compactified, you can run the following code to compactify from `idx` to `n` and then decrease `n` by 1.

```java
// Assume: Integer[] arr is the array of Integer to be compactified
// Assume: idx is the element to be removed
// Assume: n is the number of used space in arr

Integer res = arr[idx];

// Compactification: shift all elements after index idx to the left
//   to fill in the gap left by removing arr[idx]
for (int i = idx; i < n - 1; i++) {
  arr[i] = arr[i + 1];
}
arr[n - 1] = null; // removal
n = n - 1; // decrease n
```

Afterwards, you can return `res`.

!!! info "Raw Types"
    You may find yourself needing to instantiate a generic array.  But you can't, so you may need to instantiate them as a raw types instead.
    
    Consider the generic class `Pair<S, T>` again.  We cannot instantiate the following:
    
    ```java
    new Pair<String, Integer>[2];
    ```

    However, we may instantiate the raw type version of it as follows:

    ```java
    new Pair[3]
    ```

    In which case, you will get a warning similar to below if you compile with `javac -Xlint:rawtypes Pair.java`:

    ```
    _.java:_: warning: [rawtypes] found raw type: Pair
        Pair[] pArr = new Pair[2];
        ^
    missing type arguments for generic class Pair<S,T>
    where S,T are type-variables:
        S extends Object declared in class Pair
        T extends Object declared in class Pair
    _.java:_: warning: [rawtypes] found raw type: Pair
        Pair[] pArr = new Pair[2];
                        ^
    missing type arguments for generic class Pair<S,T>
    where S,T are type-variables:
        S extends Object declared in class Pair
        T extends Object declared in class Pair
    2 warnings
    ```

    In which case, you can use `@SuppressWarnings("rawtypes")` to avoid having the warning.  You still need to put the `@SuppressWarnings` in the smallest scope and comment on it.

    Additionally, `@SuppressWarnings` can be combined.  So you can have both `"unchecked"` and `"rawtypes"` suppressed as follows `@SuppressWarnings({"unchecked","rawtypes"})`.  Once you have added the necessary `@SuppressWarnings`, you should get no warning when compiling.

### Testing

You can test your `Buffer` class by running the following.  You are also encouraged to __add more tests__.

```
javac -Xlint:rawtypes -Xlint:unchecked BufferTest.java
java BufferTest
```

## Task 3: Incorporating Generic Buffer

Now that you have generic buffer, you should incorporate this to your network simulation.  There are several changes you need to make.

- `Agent::act` should now accept `Buffer<Packet>` instead of `Buffer`.
    - Changes to the `act` method should also be made to other subclasses of `Agent` if necessary.
- `Network.java` should not use raw types.

The rest of the changes is to make sure that the code compiles.

## Grading

This lab is worth 20 marks and contribute 2% to your Lab Assignment component.  The marking scheme is as follows:

| Component | Sub-Component | Marks |
|-----------|---------------|-------|
| Correctness | | 10 marks |
| | _Packet related<br>Buffer related<br>test.sh_ | _3 marks<br>3 marks<br>4 marks_ |
| OO Principles | | 10 marks |
| | _Information Hiding<br>Encapsulation<br>Polymorphism<br>Tell, Don't Ask_ | _2 marks<br>2 marks<br>3 marks<br>3 marks_ |

Correctness mark will only be awarded if your code compiles, otherwise we cannot run any tester.  While we try to give you all the possible tests, there may be some tests we missed.  You should not hardcode any test cases.

Additionally, if your code cannot compile __for any reason__, you will only get __25%__ of the mark for OO principles.  This penalty will be increased in subsequent labs.

We may make additional deductions for other issues or errors in your code such as run-time error, failure to follow instructions, errors from Lab 2 not corrected, not commenting `@SuppressWarning`, misuse of `@SuppressWarning` (_unnecessary, not in smallest scope, etc_), etc.

!!! info "Suppress Warnings"
    If you design your code correctly, you only need one `@SuppressWarnings` but you will need to include both `"unchecked"` and `"rawtypes"`.  If you have more than one, you may want to check your design again.

## Submission

To submit the lab, run the following command from the directory containing your lab 3 code.

```sh
~cs2030s/submit-lab3
```

Please check your repo after running the submission script to ensure that your files have been added correctly.  The URL to your repo is given after you run the submission script.

!!! danger "Do NOT Use Other Git Command"
    While you may be familiar with git commands, please do not use them.  Please use only the submission script `submit-labX` to ensure that your submissions are recorded properly.