# Lab 8: Keep Your World Moving

!!! abstract "Basic Information"
    - __Deadline:__ 16 November 2023, Thursday, 23:59 SST
    - __Marks:__ 20
    - __Weightage:__ 2%

!!! info "Prerequisite"
    - Caught up to [Unit 40](../40-forkjoin.md) of Online Notes.
        - Only up to [Unit 39](../39-async.md) is useful.

!!! note "Files"
    In the directory, you should see the following files:

    - __Java Files:__
        - `Computation.java`: An abstract class of computation.
        - `Parallel.java`: The parallel computation.
        - `Sequential.java`: The sequential computation.
        - `RentData.java`: A representation of a single row of rental data.
        - `Timing.java`: Timing helper class.
        - `Source.class`: The data source.

!!! warning "Errata"
    The earlier version of this incorrectly state that you cannot use loops.
    You are allowed to use loops.
    The only restriction is that the concurrency has to be done with `CompletableFuture` and not the parallel version of stream.

## Preliminary

The rental data shows the state of housing rental in Singapore.  The data is composed of 4 rows:

| Town | Block | Type | Rent |
|------|-------|------|------|
| The town name | The block name | Unit type | Monthly rental price |

While we actually have the entire data from 2021, we restrict this to only January of 2023.  Otherwise we will be dealing with millions of rows.  In fact, two towns are simplified even further to only contain a small number of blocks.

They are "BISHAN" and "QUEENSTOWN".  Most town have over 10000 rental units with over 100 blocks.  As such, running them may take minutes if not hours.  These two are simplified so that we can easily check for correctness.

!!! danger "Restriction"
    You are advised not use parallel stream and not to modify the following:

    - `Rent.csv`: Some of our computation may be order specific.
        - If you wish to understand the data, we recommend making a copy and analyze the copied data.
    - `RentData.java`: This file is given so you understand the details of the row.
        - The class file `Source.class` is compiled with the original `RentData.java`.  Changes to the `RentData.java` may cause `Source.class` to throw unexpected error.
    - `Sequential.java`: The class is given as a reference for a sequential computation.
        - The content is copied to `Parallel.java`.
        - Your task is to modify `Parallel.java` to utilize asynchronous computation.
    - `Computation.java`: This is simply an abstract class to make `Timing.java` simpler.

### Computation

The computation is performed as follows:

1. Get the name of the town.
2. Given the name of the town, find all the blocks available within the town.
    - This is done by `this.source.findBlock(town)`.
    - The available blocks within the town is returned as a `String[]`.
3. For each block, find all the unit type within the block and within the town.  This is the static method called `processBlock`.
    - This is done by `this.source.findTypeInBlock(town, block)`.
    - The available unit types is returned as a `String[]`.
4. For each type, find the minimum price.
    - This is done by `this.source.findMinPrice(town, block, type)`.
    - The minimum price for the given unit type within the given block and within the given town is returned as an `int`.

## Task: Asynchronous

The following methods can be rather expensive to compute.

- `String[] Source::findBlock(String town)`
- `String[] Source::findTypeInBlock(String town, String block)`
- `int Source::findMinPrice(String town, String block, String type)`

The given program runs the computation synchronously.  Whenever possible they should be run asynchronously.  Your task is to make both `processBlock` and `run` methods asynchronous.  Note that `processBlock` is `private` so you are allowed to modify the return type as you deem fit.

The general idea in making the program asynchronous is to utilize `CompletableFuture`.  You are to make both methods above asynchronous using `CompletableFuture`.  You may add some `private` method to simplify your work that returns a `CompletableFuture`.  With `CompletableFuture`, you simply have to take care of the task dependency.

However, note that the print order should remain the same regardless of how the concurrency is executed.  You must ensure that in the end, the print order is the same as the original sequential computation.

Some complication that you may encounter is that the number of blocks within the town as well as the number of unit types within the blocks cannot be known in advanced.  So you cannot know how many `CompletableFuture` is needed.  As such, you may need a collection of `CompletableFuture`.

Then, in the case you need to wait for all of the `CompletableFuture` to finish, remember that the method signature is

```java
static CompletableFuture<Void> allOf(CompletableFuture<?>... cfs);
```

But if you do not know how many `CompletableFuture`s are there, how can you write the code?  What else can the method above accepts?

### Guide

First, you need to determine which part can be done asynchronously and which does not benefit from concurrency or cannot be made asynchronous in the first place.  This can be obtained from dependency graph.  For those that can be run asynchronously, use the `Async` version of the available methods.  Otherwise, do **NOT** use the `Async` version or you may be penalized.

Second, although you need to make sure that the output is in the same order as sequential execution, it is possible that waiting for the first set of output takes 2s while waiting for the second set output takes only 1s.  In which case, your program may wait for 2s and then immediately prints both sets of outputs.

Alternatively, you may simply wait for all `CompletableFuture` to finish and process accordingly.  If you have kept the order, then you can process them based on the original ordering regardless of when they finish.  Note that invoking `join()` multiple times does not cause an error.  Only the first call to `join()` potentially blocks.

If you have done the concurrency correctly, you should be able to achieve about 250% speedup on PE node.  This might depend on the number of other processes running on the PE node, but the speedup should be about 200% - 400%.

You can check the speedup by running `Timing.java`.

```
javac Timing.java
java Timing
```

We default to checking on "BISHAN".  You may achieve higher speedup by modifying the input to "QUEENSTOWN".  For other towns such as "ANG MO KIO", the computation may be too long so try this at your own risk.  If it takes too long, you can press ++ctrl+c++.

```
$ javac -Xlint:rawtypes -Xlint:unchecked Parallel.java
$ bash test.sh Parallel
```

## Grading

This lab is worth 20 marks and contribute 2% to your Lab Assignment component.  The marking scheme is as follows:

| Component | Marks | Comments |
|-----------|---------------|-------|
| Correctness | 18 marks |  |
| Documentation | 2 marks |  |
| Style | _up to -4 marks_ | _Negative Marking_ |

Correctness mark will be about general correctness with manual checking on concurrency.

Most of the documentations are already done.  For any additional method you add, please provide documentations.

Additionally, if your code cannot compile __for any reason__, you will get __0 mark for the lab__.  We will check for compilation with the following command.

```bash
javac -Xlint:unchecked -Xlint:rawtypes Parallel.java
```

We may make additional deductions for other issues or errors in your code such as run-time error, failure to follow instructions, not commenting `@SuppressWarning`, misuse of `@SuppressWarning` (_unnecessary, not in smallest scope, etc_), having unnecessary conditional statements/expressions, not enough concurrency, using `Async` version unnecessarily, etc.

Note that style marks are no longer awarded. You should know how to follow the prescribed Java style by now. We will deduct up to 4 marks if there are serious violations of styles.

## Submission

To submit the lab, run the following command from the directory containing your lab 8 code.

```sh
~cs2030s/submit-lab8
```

Please check your repo after running the submission script to ensure that your files have been added correctly.  The URL to your repo is given after you run the submission script.

!!! danger "Do NOT Use Other Git Command"
    While you may be familiar with git commands, please do not use them.  Please use only the submission script `submit-labX` to ensure that your submissions are recorded properly.