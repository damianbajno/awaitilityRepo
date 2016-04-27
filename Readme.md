
Steps to run api in your project

===============================================================

1. Instaling in your project:

Java DSL

Maven:

<dependency>
      <groupId>com.jayway.awaitility</groupId>
      <artifactId>awaitility</artifactId>
      <version>1.7.0</version>
      <scope>test</scope>
</dependency>

Gradle:

 'com.jayway.awaitility:awaitility:1.7.0'

Scala DSL

Maven:

<dependency>
      <groupId>com.jayway.awaitility</groupId>
      <artifactId>awaitility-scala</artifactId>
      <version>1.7.0</version>
      <scope>test</scope>
</dependency>

SBT:

val awaitility-scala = "com.jayway.awaitility" % "awaitility-scala" % "1.7.0"

Gradle:

testCompile 'com.jayway.awaitility:awaitility-scala:1.7.0'

Groovy DSL

Maven:

<dependency>
      <groupId>com.jayway.awaitility</groupId>
      <artifactId>awaitility-groovy</artifactId>
      <version>1.7.0</version>
      <scope>test</scope>
</dependency>

Grapes:

@Grapes(
    @Grab(group='com.jayway.awaitility', module='awaitility-groovy', version='1.7.0')
) 

Gradle:

testCompile 'com.jayway.awaitility:awaitility-groovy:1.7.0'

Non-Maven / Gradle users

Download Awaitility from url:https://github.com/jayway/awaitility/wiki/Downloads and put it in your class-path. You may also need to download the third-party dependencies and put them in your classpath as well. Scala users must also download awaitility-scala and Groovy users must download awaitility-groovy.


=========================================================================

2. 

Scala documentation is on https://github.com/jayway/awaitility/wiki/Scala. Groovy documentation is on https://github.com/jayway/awaitility/wiki/Groovy.

Contents

    Static imports
    Usage examples
        Simple
        Better reuse
        Proxy based conditions
        Fields
        Atomic
        Advanced
        Java 8
        Using AssertJ or Fest Assert
        Ignoring Exceptions
        Checked exceptions in Runnable lambda expressions
    Exception handling
    Deadlock detection
    Defaults
    Polling
        Fixed
        Fibonacci
        Iterative
        Custom
    Condition Evaluation Listener
    Duration
    Important
    Links and code examples

Static imports

In order to use Awaitility effectively it's recommended to statically import the following methods from the Awaitility framework:

    com.jayway.awaitility.Awaitility.* It may also be useful to import these methods:
    com.jayway.awaitility.Duration.*
    java.util.concurrent.TimeUnit.*
    org.hamcrest.Matchers.*
    org.junit.Assert.*

Usage examples
Example 1 - Simple

Let's assume that we send a "add user" message to our asynchronous system like this:

publish(new AddUserMessage("Awaitility Rocks"));

In your test case Awaitility can help you to easily verify that the database has been updated. In its simplest form it may look something like this:

await().until(newUserIsAdded());

newUserIsAdded is a method that you implement yourself in your test case. It specifies the condition that must be fulfilled in order for Awaitility to stop waiting.

private Callable<Boolean> newUserIsAdded() {
      return new Callable<Boolean>() {
            public Boolean call() throws Exception {
                  return userRepository.size() == 1; // The condition that must be fulfilled
            }
      };
}

By default Awaitility will wait for 10 seconds and if the size of the user respository is not equal to 1 during this time it'll throw a ConditionTimeoutException failing the test. If you want a different timeout you can define it like this:

await().atMost(5, SECONDS).until(newUserWasAdded());

Example 2 - Better reuse

Awaitility also supports splitting up the condition into a supplying and a matching part for better reuse. The example above can also be written as:

await().until( userRepositorySize(), equalTo(1) );

The userRepositorySize method is now a Callable of type Integer:

private Callable<Integer> userRepositorySize() {
      return new Callable<Integer>() {
            public Integer call() throws Exception {
                  return userRepository.size(); // The condition supplier part
            }
      };
}

equalTo is a standard Hamcrest matcher specifiying the matching part of the condition for Awaitility.

Now we could reuse the userRepositorySize in a different test. E.g. let's say we have a test that adds three users at the same time:

publish(new AddUserMessage("User 1"), new AddUserMessage("User 2"), new AddUserMessage("User 3"));

We now reuse the userRepositorySize supplier and simply update the Hamcrest matcher:

await().until( userRepositorySize(), equalTo(3) );

Example 3 - Proxy based conditions

It's also possible to achieve the same result by building up the supplier part using a proxy:

await().untilCall( to(userRepository).size(), equalTo(3) );

to is method in Awaitility which you can use to define the suppling part inline in the await statement. Thus you don't need to create a Callable yourself since it's done for you. Which option you like best depends on the use case and readability.
Example 4 - Fields

You can also build the suppling part by referring to a field. E.g:

await().until( fieldIn(object).ofType(int.class), equalTo(2) );

or:

await().until( fieldIn(object).ofType(int.class).andWithName("fieldName"), equalTo(2) );

or:

await().until( fieldIn(object).ofType(int.class).andAnnotatedWith(MyAnnotation.class), equalTo(2) );

Example 5 - Atomic

If you're using Atomic structures Awaitility provides a simple way to wait until they match a specific value:

AtomicInteger atomic = new AtomicInteger(0);
// Do some async stuff that eventually updates the atomic integer
await().untilAtomic(atomic, equalTo(1));

Waiting for an AtomicBoolean is even simpler:

AtomicBoolean atomic = new AtomicBoolean(false);
// Do some async stuff that eventually updates the atomic boolean
await().untilTrue(atomic);

Example 6 - Advanced

Use a poll interval of 100 milliseconds with an initial delay of 20 milliseconds until customer status is equal to "REGISTERED". This example also uses a named await by specifying an alias ("customer registration"). This makes it easy to find out which await statement that failed if you have multiple awaits in the same test.

with().pollInterval(ONE_HUNDERED_MILLISECONDS).and().with().pollDelay(20, MILLISECONDS).await("customer registration").until(
            customerStatus(), equalTo(REGISTERED));

To use a non-fixed poll interval refer to the poll interval documentation.
Example 7 - Java 8

If you're using Java 8 you can also use lambda expressions in your conditions:

await().atMost(5, SECONDS).until(() -> userRepository.size() == 1);

Or method references:

await().atMost(5, SECONDS).until(userRepository::isNotEmpty);

Or a combination of method references and Hamcrest matchers:

await().atMost(5, SECONDS).until(userRepository::size, is(1));

For examples refer to the Jayway team blog.
Example 8 - Using AssertJ or Fest Assert

Starting from version 1.6.0 you can use AssertJ or Fest Assert as an alternative to Hamcrest (it's actually possible to use any third party library that throws exception on error). For example:

await().atMost(5, SECONDS).until(() -> assertThat(fakeRepository.getValue()).isEqualTo(1));

Example 9 - Ignoring Exceptions

As of Awaitility 1.6.5 you can choose to ignore exceptions while evaluating a condition. This is useful if you're waiting for something that throws exceptions as an intermediate state before the final state is reached. As an example take Spring's SocketUtils class that allows you to find an TCP port in a given range. It will throw an exception if no port is available in the given range. So let's say we know that some ports in a given range are not available but we want to wait for them to be so. This is an example where we may choose to ignore any exception thrown by SocketUtils. For example:

given().ignoreExceptions().await().until(() -> SocketUtils.findAvailableTcpPort(x,y));

This instruct Awaitility to ignore all caught exceptions during condition evaluation. Exceptions will be treated as evaluating to false. The test will not fail (unless it times out) upon an exception matching the supplied exception type. You can also ignore specific exceptions:

given().ignoreException(IllegalStateException.class).await().until(() -> SocketUtils.findAvailableTcpPort(x,y));

or using a Hamcrest matcher:

given().ignoreExceptionsMatching(instanceOf(RuntimeException.class)).await().until(() -> SocketUtils.findAvailableTcpPort(x,y));

or a predicate (Java 8):

given().ignoreExceptionsMatching(e -> e.getMessage().startsWith("Could not find an available")).await().until(something());

Example 10 - Checked exceptions in Runnable lambda expressions

A Runnable interface in Java doesn't allow you to throw checked exceptions. So if you have a method like this:

public void waitUntilCompleted() throws Exception { ... }

that may throw an exception, you also have to catch this exception when using lambda expressions:

await().until(() -> {
   try {
      waitUntilCompleted();
   } catch(Exception e) {
     // Handle exception
   }
});

This is very verbose so Awaitility 1.7.0 introduces a helper method called matches that can be statically imported from the com.jayway.awaitility.Awaitility class. Wrapping the lambda expression in this function allows you to do:

await().until(matches(() -> waitUntilCompleted()));

which significantly reduces the boiler-plate code and improves readability.
Exception handling

By default Awaitility catches uncaught exceptions in all threads and propagates them to the awaiting thread. This means that your test-case will indicate failure even if it's not the test-thread that threw the uncaught exception.

You can choose to ignore certain exceptions, see here.
Deadlock detection

Awaitility 1.6.2 automatically detect deadlocks and associates the cause of the ConditionTimeoutException with a DeadlockException. The DeadlockException contains information on what threads are causing the deadlock.
Defaults

If you don't specify any timeout Awaitility will wait for 10 seconds and then throw a ConditionTimeoutException if the condition has not been fulfilled. The default poll interval and poll delay is 100 milliseconds. You can also specify the default values yourself using:

  Awaitility.setDefaultTimeout(..)
  Awaitility.setDefaultPollInterval(..)
  Awaitility.setDefaultPollDelay(..)

You can also reset back to the default values using Awaitility.reset().
Polling

Awaitility 1.7.0 introduced non-fixed poll intervals complementary to the fixed poll interval available in the previous version.

Note that since Awaitility uses polling to verify that a condition matches it's not recommended to use it for precise performance testing. In these cases it's better to use an AOP framework such as AspectJ's compile-time weaving.

Also note that Duration.ZERO is used as start value for all non-fixed poll intervals. For fixed poll intervals the poll delay is equal to the duration of the FixedPollInterval for backward compatible reasons.

Have a look at this blog for additional details.
Fixed Poll Interval

This is the default poll interval mechanism of Awaitilty. When using the DSL in the normal way such as:

with().pollDelay(100, MILLISECONDS).and().pollInterval(200, MILLISECONDS).await().until(<condition>);

Awaitility will use a FixedPollInterval. This means that Awaitility will check if the specified condition is fulfilled for the first time after a "poll delay" (the initial delay before the polling begins, 100ms in the example above). Unless explicitly specified, Awaitility will use the same poll delay as poll interval (note that this is only true of fixed poll intervals like in the example above). This means that it checks the condition periodically first after the given poll delay, and subsequently with the given poll interval; that is conditions are checked after pollDelay then pollDelay+pollInterval, then pollDelay + (2 * pollInterval), and so on. If you change the poll interval the poll delay will also change to match the specified poll interval unless you've specified a poll delay explicitly.
Fibonacci Poll Interval

The FibonacciPollInterval generates a non-linear poll interval based on the fibonacci sequence. Usage example:

with().pollInterval(fibonacci()).await().until(..);

where fibonacci is statically imported from com.jayway.awaitility.pollinterval.FibonacciPollInterval. This will generate a poll interval of 1, 1, 2, 3, 5, 8, 13, .. milliseconds. You can configure the time unit to use, for example seconds instead of milliseconds:

with().pollInterval(fibonacci(TimeUnit.SECONDS)).await().until(..);

Or a bit more english-like:

with().pollInterval(fibonacci().with().timeUnit(SECONDS).and().offset(5)).await().until(..);

Offset means that the fibonacci sequence is started from this offset (by default the offset is 0). The offset can also be negative (-1) to start with 0 (fib(0) = 0).
Iterative Poll Interval

A poll interval that is generated by a function and a start duration. The function is free to do anything with the duration. For example:

await().with().pollInterval(iterative(duration -> duration.multiply(2)), Duration.FIVE_HUNDRED_MILLISECONDS).until(..);

or a bit more english-like:

await().with().pollInterval(iterative(duration -> duration.multiply(2)).with().startDuration(FIVE_HUNDRED_MILLISECONDS)).until(..);

This generates a poll interval sequence that looks like this (ms): 500, 1000, 2000, 4000, 8000, 16000, ...

Note that if you specify a poll delay this delay will take place before the first poll interval is generated by this poll interval. See javadoc for more info.
Custom Poll Interval

It's possible to roll your own poll interval by implementing the PollInterval interface. This is a functional interface so in Java 8 you can just do like this:

await().with().pollInterval((__, previous) -> previous.multiply(2).plus(1)).until(..);

In this example we create a PollInterval that is implemented as a (bi-) function that takes the previous poll interval duration and multiplies it by 2 and adds 1. __ just signals that we don't care about the poll count that is also provided by the PollInterval interface. Poll count is required when creating poll intervals that are not (only) interested in the previous duration but rather generates its duration based on the number of times it has been called. For example the FibonacciPollInterval uses only the poll count:

await().with().pollInterval((pollCount, __) -> new Duration(fib(pollCount), MILLISECONDS)).until(..);

I most cases it shouldn't be necessary to implement a poll interval from scratch. Supply a function to the IterativePollInterval instead.
Condition Evaluation Listener

Awaitility 1.6.1 introduced the concept of Condition Evaluation Listener. This can be used to get information each time a condition has been evaluated by Awaitility. You can for example use this to get the intermediate values of a condition before it is fulfilled. It can also be used for logging purposes. For example:

with().
         conditionEvaluationListener(condition -> System.out.printf("%s (elapsed time %dms, remaining time %dms)\n", condition.getDescription(), condition.getElapsedTimeInMS(), condition.getRemainingTimeInMS())).
         await().atMost(Duration.TEN_SECONDS).until(new CountDown(5), is(equalTo(0)));

will print the following to the console:

    com.jayway.awaitility.AwaitilityJava8Test$CountDown expected (<0> or a value less than <0>) but was <5> (elapsed time 101ms, remaining time 1899ms)
    com.jayway.awaitility.AwaitilityJava8Test$CountDown expected (<0> or a value less than <0>) but was <4> (elapsed time 204ms, remaining time 1796ms)
    com.jayway.awaitility.AwaitilityJava8Test$CountDown expected (<0> or a value less than <0>) but was <3> (elapsed time 306ms, remaining time 1694ms)
    com.jayway.awaitility.AwaitilityJava8Test$CountDown expected (<0> or a value less than <0>) but was <2> (elapsed time 407ms, remaining time 1593ms)
    com.jayway.awaitility.AwaitilityJava8Test$CountDown expected (<0> or a value less than <0>) but was <1> (elapsed time 508ms, remaining time 1492ms)
    com.jayway.awaitility.AwaitilityJava8Test$CountDown reached its end value of (<0> or a value less than <0>) (elapsed time 610ms, remaining time 1390ms)

There's a built-in ConditionEvaluationListener for logging called ConditionEvaluationLogger that you can use like this:

with().conditionEvaluationListener(new ConditionEvaluationLogger()).await(). ...

Note that Awaitility 1.6.1 only support Hamcrest-based conditions but in 1.6.2 and above all condition types work.
Duration

Awaitility provides a Duration class that contains some predefined duration values such as ONE_HUNDRED_MILLISECONDS, FIVE_SECONDS and ONE_MINUTE. You can also perform some basic math operations on a Duration instance. For example:

new Duration(5, SECONDS).plus(17, MILLISECONDS);

which will return a new Duration of 5017 milliseconds. Note that Durations are immutable so calling plus will return a new instance. This is mainly useful when working with non-fixed poll intervals.
Important

Awaitility does nothing to ensure thread safety or thread synchronization! This is your responsibility! Make sure your code is correctly synchronized or that you are using thread safe data structures such as volatile fields or classes such as AtomicInteger and ConcurrentHashMap.
Links and code example.

