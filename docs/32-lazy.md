# Unit 32: Lazy Evaluation

After this unit, students should understand:

- what is lazy evaluation and how lambda expression allows us to delay the execution of a computation
- how memoization and the `Lazy<T>` abstraction allows us to evaluate an expression exactly once.

## Lambda as Delayed Data

When we write a lambda expression like this:
```Java
Transformer<Integer, Integer> f = x -> x + 1;
```

we are just defining an expression.  We are not invoking the function `x + 1`.  This is perhaps clear to most students since to invoke the function, we need an argument for `x`, and no argument is supplied when we define `f`.

Consider the following functional interfaces instead:
```Java
@FunctionalInterface
interface Producer<T> {
  T produce();
}

@FunctionalInterface
interface Task {
  void run();
}
```

These functional interfaces have a method that does not take in a parameter.  So, we would be using them like such:

```Java
i = 4;
Task print = () -> System.out.println(i);
Producer<String> toStr = () -> Integer.toString(i);
```

Keep in mind that these are still lambda expressions and nothing is executed by simply declaring them.  We are just saving them to be executed later.

Lambda expression, therefore, allows us to delay the execution of code, saving them until we need it later.  This enables another powerful mechanism called _lazy evaluation_.  We can build up a sequence of complex computations, without actually executing them, until we need to.  Expressions are evaluated on demand when needed.

Consider the following class:

```Java
// Version 0.1 (eager evaluation)

class Logger {
  enum LogLevel { INFO, WARNING, ERROR };

  public static LogLevel currLogLevel = LogLevel.WARNING;

  static void log(LogLevel level, String msg) {
    if (level.compareTo(Logger.currLogLevel) >= 0) {
      System.out.println(" [" + level + "] " + msg);
    }
  }
}
```

The `log` method checks the seriousness level of the message against the current log level, and only prints the message if the level of the message is the same or higher.  For instance, if the current log level is `WARNING`, then

```Java
Logger.log(Logger.LogLevel.INFO, 
    "User " + System.getProperty("user.name") + " has logged in");
```

will not get printed.

However, regardless of whether the log message will be printed, the method `System.getProperty("user.name")` will be evaluated, which is wasteful.

A better design for this case is to wrap the message `msg` within a lambda expression, so that it does not get evaluated eagerly when we pass it in as a parameter.  We can wrap the message with a `Producer<String>`.  The new `lazyLog` method would look like this:

```Java
// Version 0.2 (with Producer)

class Logger {
  enum LogLevel { INFO, WARNING, ERROR };

  public static LogLevel currLogLevel = LogLevel.WARNING;

  static void lazyLog(LogLevel level, Producer<String> msg) {
    if (level.compareTo(Logger.currLogLevel) >= 0) {
	  System.out.println(" [" + level + "] " + msg.produce());
    }
  }
}
```

and is invoked like this:
```Java
Logger.lazyLog(Logger.LogLevel.INFO, 
    () -> "User " + System.getProperty("user.name") + " has logged in");
```

The method `System.getProperty("user.name")` is now lazily called, only if the message is going to be printed.

## Memoization

We have so far seen one way of being lazy, i.e., procrastinating our computation until we really need the data.  Another way of being lazy is not to repeat ourselves.  If we have computed the value of a function before, we can cache (or memoize) the value, keep it somewhere, so that we don't need to compute it again.  This is useful, of course, only if the function is pure &mdash; regardless of how many times we invoke the function, it always returns the same value, and invoking it has no side effects on the execution of the program.  Here, we see another important advantage of keeping our code pure and free of side effects &mdash; so that we can be lazy!

While other languages such as Scala as native support for lazy variables, Java does not.  So let's build a simple one here.  (You will build a more sophisticated one in Lab 6) 

```Java
class Lazy<T> {
  private T value;
  private boolean evaluated;
  private Producer<T> producer;

  public Lazy(Producer<T> producer) {
    evaluated = false;
	value = null;
	this.producer = producer;
  }

  public T get() {
	if (!evaluated) {
	  value = producer.produce();
	  evaluated = true;
	}
	return value;
  }
}
```

We can now rewrite our `Logger` as

```Java
// version 0.3 (with Lazy)

class Logger {
  enum LogLevel { INFO, WARNING, ERROR };

  public static LogLevel currLogLevel = LogLevel.WARNING;

  static void lazyLog(LogLevel level, Lazy<String> msg) {
    if (level.compareTo(Logger.currLogLevel) >= 0) {
	  System.out.println(" [" + level + "] " + msg.get());
    }
  }
}
```

and call it with:
```Java
Lazy<String> loginMessage = new Lazy(
    () -> "User " + System.getProperty("user.name") + " has logged in");
Logger.lazyLog(Logger.LogLevel.INFO, loginMessage);
```

If `loginMessage` is used in multiple places, memoization ensures that `System.getProperty("user.name")` and the concatenation of the strings are done only once.
