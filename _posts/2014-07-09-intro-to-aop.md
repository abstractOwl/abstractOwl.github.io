---
layout: post
title:  "Intro to Aspect-Oriented Programming"
---

**UPDATE**: The kind folks at [POSTD](http://www.postd.cc) have translated
this post into Japanese! You can find it
[here](http://postd.cc/intro-to-aop/).

---

In any non-trivial application, functions quickly become weighed down with
supporting functionality like logging,
[tracing](https://en.wikipedia.org/wiki/Tracing_(software)), and metrics
gathering. These blocks of code tend to echo throughout codebases with low
variance and are referred to as **cross-cutting concerns**.
[Aspect-Oriented Programming](https://en.wikipedia.org/wiki/Aspect-oriented_programming)
(AOP) helps achieve the
[separation of concerns](https://en.wikipedia.org/wiki/Separation_of_concerns)
by pulling these blocks into modules called aspects.


## AOP in Practice

### Introducing the Phone class[^1]

At the risk of presenting a relatively contrived example, I have constructed a
simple Phone class with a single method `dial`.

    function Phone() {};
    Phone.prototype.dial = function (friend) {
        var start = new Date().getTime();
        log('dial(): {}', friend.name);

        if (friend && friend.picksUp) {
            var end = new Date().getTime();
            log('Call lasted: {} milliseconds', end - start);
            return friend.message;
        } else {
            log('Error: No answer!');
            throw 'No answer!';
        }
    };

The `dial` function works as follows:

1. Log a tracing message containing the method name and friend's name
2. If the friend picks up:
    1. Log the call duration
    2. Return the friend's message
3. Otherwise:
    1. Create a log message indicating the friend failed to pick up
    2. Throw a 'No answer!' exception

Remember that AOP aims to separate concerns from business logic. Now look back
at the Phone class. How much of it is actually business logic?

1. <strike>Log a debug message with the method name and friend's name</strike>
2. If the friend picks up:
    1. <strike>Log the call duration</strike>
    2. Return the friend's message
3. Otherwise:
    1. <strike>Create a log message indicating the friend failed to pick
        up</strike>
    2. Throw a 'No answer!' exception

It turns out that nearly half our function code is devoted to concerns! Let's
see how to separate these concerns into aspects with the
[meld](https://github.com/cujojs/meld) AOP library.


### Adding Callee Metadata

To start off, let's pull out the initial tracing statement into a piece of
advice. **Advice** refers to a function that acts upon another function. Here,
we create a `before` advice, an advice that runs before the execution of
other functions.

    meld.before(Phone.prototype, 'dial', function () {
        log(
            '{}: Calling {}... ',
            meld.joinpoint().method,
            meld.joinpoint().args[0].name
        );
    });

We use the `meld.before` method to apply the log advice before `Phone.dial`
runs. In AOP terminology, we apply the log advice at the `before`
**join point** of the `Phone.dial` **pointcut**. The join point is the point in
execution where the advice is applied, while the pointcut defines which
functions the advice is applied on.

Notice that join points provide relevant information about the associated
pointcut. In this aspect, we access `meld.joinpoint().method` and
`meld.joinpoint().args[0].name` to access the method name and arguments.
Join points also provide other useful meta features, as we will see in the next
section.

Tracing is an extremely common use case for AOP. It allows developers to easily
observe the passage of values through an application without modifying any
class code.


### Timing the Call

Next, we extract the timing function:

    meld.around(Phone.prototype, 'dial', function (joinPoint) {
        var start = new Date().getTime();
        var message = joinPoint.proceed();
        var end = new Date().getTime();
        log('Call lasted: {} milliseconds!', (end - start));
        return joinPoint.args[0].name + ': ' + message;
    });

As the name implies, `around` advice is executed *around* the function being
called. That is, it contains code that is run both before and after the
function call. This special join point has a `.proceed` function that tells
meld to go ahead and run the function this aspect is associated with. If
`joinPoint.proceed` is not used, `phone.dial` will never be executed.

    // Calling fn2 from fn1, where fn2 has around advice
    ----------            -----------                   ---------
    |        | --fn2()--> |         |                   |       |
    | fn1()  |            | around  | --jp.proceed()--> | fn2() |
    |        |            |  advice | <----return-------|       |
    |        | <-return-- |         |                   |       |
    ----------            -----------                   ---------

`around` also allows us to change the return value of the function. The return
value of `joinPoint.proceed` is the return value of the function call.
`around`'s advice can modify it freely before returning it to the caller, or
decide to return a completely different value altogether. The `around` advice
essentially serves as a [proxy](https://en.wikipedia.org/wiki/Proxy_pattern)
between the caller and the callee.

`around` is the most powerful and robust join point in AOP due to its control
over the callee function.


### Error Handling

Finally, let's look at logging exceptions. The `afterThrowing` aspect is
executed after an exception is thrown.

    meld.afterThrowing(Phone.prototype, 'dial', function (error) {
        log('Error: {}', error);
    });

Here, the join point passes the thrown exception to the `afterThrowing` advice.
`afterThrowing` is particularly useful for collecting metrics about where/why
an application is encountering errors. You can think of a web server
aggregating data on which URLs many visitors are encountering "404 Not Found"
errors at.

In more complex applications, this join point can also be used to implement
retry logic for code that might experience transient errors:

    meld.afterThrowing(SomeObject, 'someMethod', function (error) {
        if (isTransient(error)) { // Check whether error is recoverable
            setTimeout(function () {
                // Retry after a delay
                SomeObject.someMethod();
            }, delay);
        }
    });

The `afterThrowing` aspect can be handy for logging and handling exceptions.


## Putting It All Together

The finished code can be seen on [JSFiddle](http://jsfiddle.net/G2e22/) or
[GitHub](https://gist.github.com/abstractOwl/dbcc8ee5f9ac61323d33). As you can
see, using AOP allowed us to condense our function down to 5 lines of business
logic and 3 reusable aspects.

While here we have stuck all the code in one file, developers with larger
codebases may choose to split the aspects into a separate file.


## Parting Thoughts

Aspect-Oriented Programming can be useful for refactoring applications. In
addition to the types of advice discussed, AOP libraries also provide:

* `after` advice: Run after function call, regardless of success.
* `afterReturning` advice: Run after successful function call.

In the writing of this tutorial, I chose to use Javascript due to its
popularity and convenience, but AOP is available in many other languages.
Java, for example, has [Spring AOP](http://spring.io) and
[AspectJ](http://eclipse.org/aspectj). Both libraries, in addition to matching
the functionality of meld.js, also support greater pointcut flexibility and
reuse. There are Spring AOP snippets in my
[snippets](https://github.com/abstractOwl/snippets/tree/master/java/HelloAspects)
repository on GitHub for those interested.

Aspect-Oriented Programming provides powerful techniques to decouple concerns
from business logic, resulting in cleaner code.


## Further Reading

1. [meld on Github](https://github.com/cujojs/meld)

2. [AOP on Wikipedia](https://en.wikipedia.org/wiki/Aspect-oriented_programming)

3. [Every time you use CSS, you’re doing Aspect-Oriented Programming](http://plpatterns.com/post/482063133/every-time-you-use-css-youre-doing-aspect-oriented)


## Footnotes

[^1]: *Technically*, Javascript has "prototypes", not "classes".
