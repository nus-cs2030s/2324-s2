# Unit 38: Threads

After this unit, students should:

- understand the behaviour of thread
- be able to create a simple thread

## Synchronous Programming

So far, when we invoke a method in Java, we expect the method to return us a value when it is done.  If the method is not done, the execution of our program stalls, waiting for the method to complete its execution.  Only after the method returns can the execution of our program continue.

We say that the method _blocks_ until it returns.   Such a programming model is known as _synchronous programming_.

Synchronous programming is not very efficient, especially when there are frequent method calls that block for a long period (such as methods that involve expensive computations or reading from a remote server over the Internet).

What if we want our program to do something while we wait for the method to return?  For instance, refreshing the UI, or do other computations?

## Threads

One way to achieve this is to use _threads_.  A thread is a single flow of execution in a program.  Since the beginning of this module, we have been writing single-thread programs, except for parallel streams in Unit 37.

Java provides a class called [`java.lang.Thread`](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Thread.html) that we can use to encapsulate a function to run in a separate thread.  The following example how we can create and run two threads:

```Java
new Thread(() -> {
  for (int i = 1; i < 100; i += 1) {
    System.out.print("_");
  }
}).start();

new Thread(() -> {
  for (int i = 2; i < 100; i += 1) {
    System.out.print("*");
  }
}).start();
```

The `new Thread(..)` is our usual constructor to create a `Thread` instance.  The constructor takes a `Runnable` instance as an argument.  A `Runnable` is a functional interface with a method `run()` that takes in no parameter and returns `void`.

With each `Thread` instance, we run `start()`, which causes the given lambda expression to run.  Note that `start()` returns immediately.  It _does not return_ only after the given lambda expression completes its execution.  This property differs from what we are used to, where a method blocks until the task given completes.   This is known as _asynchronous_ execution.

The two threads above now run in two separate sequences of execution.  The operating system has a scheduler that decides which threads to run when, and on which core (or which processor).  You might see different interleaving of executions every time you run the same program.

Java provides more than one way to create a thread.  The `Thread` class also contains methods that we can use to query and control, in a finer grain manner, how the thread could be executed.

### Names

Every thread in Java has a name, printing out its name is useful for peeking under the hood to see what is happening.  We can use the instance method `getName()` to find out the name of a thread, and the class method `Thread.currentThread()` to get the reference of the current running thread.

```Java
System.out.println(Thread.currentThread().getName());
new Thread(() -> {
  System.out.print(Thread.currentThread().getName());
  for (int i = 1; i < 100; i += 1) {
    System.out.print("_");
  }
}).start();

new Thread(() -> {
  System.out.print(Thread.currentThread().getName());
  for (int i = 2; i < 100; i += 1) {
    System.out.print("*");
  }
}).start();
```

Note that the above will also print the name of the thread called `main`, which is a thread created automatically for us every time our program runs and the class method `main()` is invoked.

With this method, you can now "visualize" how many parallel threads are created when you invoke a parallel stream.

Try
```Java
Stream.of(1, 2, 3, 4)
      .parallel()
      .reduce(0, (x, y) -> { 
        System.out.println(Thread.currentThread().getName()); 
        return x + y; 
      });
```

and you will see something like this:
```
main
ForkJoinPool.commonPool-worker-5
ForkJoinPool.commonPool-worker-5
ForkJoinPool.commonPool-worker-9
ForkJoinPool.commonPool-worker-3
ForkJoinPool.commonPool-worker-3
ForkJoinPool.commonPool-worker-3
```

being printed.  This shows four concurrent threads running to reduce the stream of 1, 2, 3, 4 (including `main`).

If you remove the `parallel()` call, then only `main` is printed, showing the reduction being done sequentially in a single thread.

```Java
Stream.of(1, 2, 3, 4)
      .reduce(0, (x, y) -> { 
        System.out.println(Thread.currentThread().getName()); 
        return x + y; 
      });
```

### Sleep

Another useful method in the `Thread` class is `sleep`.  You can cause the current execution thread to pause execution immediately for a given period (in milliseconds).   After the sleep timer is over, the thread is ready to be chosen by the scheduler to run again.

The following code prints a `"."` on-screen every second while another expensive computation is running.

```Java
Thread findPrime = new Thread(() -> {
  System.out.println(
	  Stream.iterate(2, i -> i + 1)
          .filter(i -> isPrime(i))
          .limit(1_000_000L)
          .reduce((x, y) -> y)
          .orElse(null));
});

findPrime.start();

while (findPrime.isAlive()) {
  try {
    Thread.sleep(1000);
    System.out.print(".");
  } catch (InterruptedException e) {
    System.out.print("interrupted");
  }
} 
```

In our examples, we often use `Thread.sleep()` in our methods to pretend that we are working hard on expensive computation to keep our examples simple.

Two more things to note:

- The example above shows how we use `isAlive()` to periodically check if another thread is still running.
- The program exits only after all the threads created run to their completion.

