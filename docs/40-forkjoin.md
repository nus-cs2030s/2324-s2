# Unit 40: Fork and Join

After this unit, students should:

- understand the task deque and work stealing
- understand the behaviour of `fork` and `join` (and `compute`)
- be able to order `fork` and `join` efficiently
- be able to use `RecursiveTask`

## Thread Pool

We now look under the hood of parallel `Stream` and `CompletableFuture<T>` to explore how Java manages its threads.  Recall that creating and destroying threads is not cheap, and as much as possible we should reuse existing threads to perform different tasks.  This goal can be achieved by using a _thread pool_. 

A thread pool consists of (i) a collection of threads, each waiting for a task to execute, and (ii) a collection of tasks to be executed.  Typically the tasks are put in a _shared queue_, and an idle thread picks up a task from the shared queue to execute.

To illustrate this concept, here is a trivial thread pool with a single thread:

```Java
Queue<Runnable> queue;
new Thread(() -> {
  while (true) {
	if (!queue.isEmpty()) {
	  Runnable r = queue.dequeue();
	  r.run();
    }
  }
}).start();

for (int i = 0; i < 100; i++) {
  int count = i;
  queue.add(() -> System.out.println(count));
}
```

We assume that `Queue<T>` can be safely modified concurrently (i.e., it is thread-safe) in the sample code above.  Otherwise, just like the example you have seen in [parallel streams](37-parallel.md) with `List`, items might be lost.

## Fork and Join

Java implements a thread pool called `ForkJoinPool` that is fine-tuned for the fork-join model of recursive parallel execution.  

The Fork-join model is essentially a parallel divide-and-conquer model of computation.  The general idea for the fork-join model is to solve a problem by breaking up the problem into identical problems but with smaller size (_fork_), then solve the smaller version of the problem recursively, then combine the results (_join_).   This repeats recursively until the problem size is small enough &mdash; we have reached the base case and so we just solve the problem sequentially without further parallelization.

In Java, we can create a task that we can fork and join as an instance of abstract class `RecursiveTask<T>`.  `RecursiveTask<T>` supports the methods `fork()`, which submits a smaller version of the task for execution, and `join()` (which waits for the smaller tasks to complete and return).   `RecursiveTask<T>` has an abstract method `compute()`, which we, as the client, have to define to specify what computation we want to compute.

Here is a simple `RecursiveTask<T>` that recursively sums up the content of an array:
```Java
class Summer extends RecursiveTask<Integer> {
  private static final int FORK_THRESHOLD = 2;
  private int low;
  private int high;
  private int[] array;

  public Summer(int low, int high, int[] array) {
    this.low = low;
    this.high = high;
    this.array = array;
  }

  @Override
  protected Integer compute() {
    // stop splitting into subtask if array is already small.
    if (high - low < FORK_THRESHOLD) {
      int sum = 0;
      for (int i = low; i < high; i++) {
        sum += array[i];
      }
      return sum;
    }

    int middle = (low + high) / 2;
    Summer left = new Summer(low, middle, array);
    Summer right = new Summer(middle, high, array);
    left.fork();
    return right.compute() + left.join();
  }
}
```

To run this task, we run:
```Java
Summer task = new Summer(0, array.length, array);
int sum = task.compute();
```

The line `task.compute()` above is just like another method invocation.  It causes the method `compute()` to be invoked, and if the array is big enough, two new `Summer` instances, `left` and `right`, to be created.  `left`.  We then call `left.fork()`, which adds the tasks to a thread pool so that one of the threads can call its `compute()` method.  We subsequently call `right.compute()` (which is a normal method call).  Finally, we call `left.join()`, which blocks until the computation of the recursive sum is completed and returned.  We add the result from `left` and `right` together and return the sum.

There are other ways we can combine and order the execution of `fork()`, `compute()`, and `join()`.  Some are better than others.  We will explore more in the exercises.

## `ForkJoinPool`

Let's now explore the idea behind how Java manages the thread pool with fork-join tasks.  The details are beyond the scope of this module, but it would be interesting to note a few key points, as follows:

- Each thread has a deque[^1] of tasks.  
- When a thread is idle, it checks its deque of tasks.  If the deque is not empty, it picks up a task at the head of the deque to execute (e.g., invoke its `compute()` method).  Otherwise, if the deque is empty, it picks up a task from the _tail_ of the deque of another thread to run.  The latter is a mechanism called _work stealing_.
- When `fork()` is called, the caller adds itself to the _head_ of the deque of the executing thread.  This is done so that the most recently forked task gets executed next, similar to how normal recursive calls.
- When `join()` is called, several cases might happen.  If the subtask to be joined hasn't been executed, its `compute()` method is called and the subtask is executed.  If the subtask to be joined has been completed (some other thread has stolen this and completed it), then the result is read, and `join()` returns.  If the subtask to be joined has been stolen and is being executed by another thread, then the current thread finds some other tasks to work on either in its local deque or steal another task from another deque.

[^1]: A deque is a double-ended queue.  It behaves like both stack and queue.

The beauty of the mechanism here is that the threads always look for something to do and they cooperate to get as much work done as possible.

The mechanism here is similar to that implemented in .NET and Rust.

## Order of `fork()` and `join()`

One implication of how `ForkJoinPool` adds and removes tasks from the deque is the order in which we call `fork()` and `join()`.  Since the most recently forked task is likely to be executed next, we should `join()` the most recent `fork()` task first.  In other words, the order of forking should be the reverse of the order of joining.

In the class `Summer` above,
```java
    left.fork();  // >-----------+
    right.fork(); // >--------+  | should have
    return right.join() // <--+  | no crossing
         + left.join(); // <-----+
```

is more efficient than
```java
    left.fork();  // >-------------+
    right.fork(); // >----------+  | there is crossing
    return left.join()   // <---|--+
         + right.join(); // <---+
```

In other words, your `fork()`, `compute()`, `join()` order should form a [_palindrome_](https://en.wikipedia.org/wiki/Palindrome) and there should be no crossing.  Additionally, there should only be at most a single `compute` and it should be in the middle of the palindrome.

For example, the following is ok.
```java
    left.fork();  // >-----------+
    return right.compute() //    | compute in middle
         + left.join(); // <-----+
```

But the following is not.
```java
    return left.compute()   // this is practically
         + right.compute(); // not even concurrent
```