---
layout: post
title: "Dabbling in Erlang, part 1: Hello function"
date:  2013-05-19
---


Recently I've been dabbling in Erlang and I'm enjoying it a lot. I decided to start a series of posts in which I'll address things that I believe are worth mentioning.

## Meet Erlang
[Erlang](http://en.wikipedia.org/wiki/Erlang_%28programming_language%29) is a functional, concurrent, dynamically-typed language initially developed at Ericsson around 1986 in order to power its telecom infrastructure. 

It is a great fit when scalability, availability and distribution are crucial (exactly what telecom applications need). This greatness stems from Erlang's concurrency model, its error-handling capabilities and the OTP framework.

Before I dive into the language, I'd like to talk a little about functional programming in general. In fact, I've just decided that this will be the subject of this post.

## How I stopped worrying about state and loved the function
Coming to a declarative language like Erlang from an imperative language like Ruby certainly requires a mental switch:

You'll have to stop thinking in terms of mutating the program's state and start thinking in terms of functions and data. Your programs will no more describe *how* something should be done but *what* should be done. You'll have to wave goodbye to your lovely side-effects. There are no things like `a += 1` here.

Where there used to be `for` loops, there is *recursion* instead.

This:
{% highlight ruby %}
sum = 0

for i in [1, 2, 3, 4]
  sum += i
end

sum
{% endhighlight %}

becomes this:

{% highlight erlang %}
sum([])    -> 0;
sum([H|T]) -> H + sum(T).

sum([1, 2, 3, 4]).
{% endhighlight %}

In the second version there's no mutation at all. We just define a recursive function which gets in some data and spits out some other data, without messing with the outside world at all.

That's what functional programming is all about: functions spitting out data, isolated in their own little world, without touching anything outside.

But what benefits does the absence of side-effects grants us?

### Programs are easier to reason about 
Since there are no side-effects, it tends to be easier to reason about the code. In order to understand what a function does in Erlang, you only have to read its code and nothing else. This most times is in contrast with imperative programs, where you may have to read code in a lot of different places to have a complete picture of what a function is doing.

### Easier debugging
This in turn leads to programs which are easier to debug, since you can immediately trace the source of bugs that occur. Add single assignment to the mix and debugging becomes a lot simpler. All you have to do is glance at the stack-trace.

### Transparent shift to multicore
It's well known that parallelizing code where there is shared mutable state is hard. You have to mess with locks and mutexes and then bad things will eventually happen: deadlocks, race-conditions etc. Having immutable state and no shared memory has enormous advantages when you want to make your program run in parallel. All you have to do is structure your code concurrently and then the shift to multicore is transparent.

### Easy to test
Since functions are deterministic and the result totally depends on the input and nothing else, it means that if they produce the same answer today, they will produce the same answer tomorrow. Combine this with the fact that there are no external dependencies, and testing becomes a lot easier. Remember what we were taught in school: if `f(5) = 10`, it means f(5) will *always* be 10. It's data-in, data-out, simple.

## So why hasn't functional programming taken over yet?
Really, why we see a lot of C, Java, Ruby, Python and not so much Haskell, F#, Clojure, Erlang? There must be a tradeoff; there always is.

People say functional programming will never dominate the industry like imperative does right now, since its advantages do no  justify spending the resources for such a big switch.

Some say this is exactly what's happening right now. The last 4-5 years there's been a buzz around functional languages you can see them more and more used in real-world applications. Erlang is used by [Facebook, Amazon, Yahoo, Ericsson, Github](http://stackoverflow.com/questions/1636455/where-is-erlang-used-and-why). Scala is used by [Twitter](http://blog.redfin.com/devblog/2010/05/how_and_why_twitter_uses_scala.html). [Clojure](http://www.infoq.com/presentations/Clojure-powered-Startups) powers some nice start-ups ([Prismatic](http://getprismatic.com) is my favorite).

However, it's only in the recent years that such languages gained massive popularity. Why is that? 

To begin with, side-effects can be as much a disadvantage as it can be an advantage. It all depends in the problem you're facing. Truth is, in most real-world problems you want to have side-effects: when users of your web application click on a button to make the purchase, it just makes more sense to alter their current balance by doing something like `balance -= 50`, than to construct a new balance with the new state. You also want to persist their updated balances to the database.

Also, thinking in a functional manner and reading functional code is a difficult skill and requires quite a mental shift. This is partly due to the the fact that when we learn programming in schools, universities or whatever, we are taught to think in an imperative way and that's because until now the IT industry is mostly dominated by imperative languages (see Java, Ruby, Python) and this may be because there are not such people doing functional programming. The chicken-egg problem. 

Last, when big amounts of money are involved, you can't discard the current working system which pays the debts and build a new one if the benefits do not justify this change. "Functional programming is cool" is not a good reason. 

This has been said a thousand times but it really boils down to this: *There is no silver-bullet*. It all depends on the problem you're facing. I should quote [Zed Shaw](http://learnpythonthehardway.org/book/advice.html) at this point:

> Do not get sucked into the religion surrounding programming languages as that will only blind you to their true purpose of being your tool for doing interesting things. --Zed Shaw

So why not mix and match? The bigger you get, the more problems you have to solve, so the more tools you'll need.

Learning a functional programming language is one of the greatest challenges you can take. You'll train different areas of your brain, you will gain perspective, expand your skills and will become a better programmer.

In the next post I'll dive into the language and talk about its strong points.

------------------

The [second part](/2013/10/12/dabbling-in-erlang-a-minimal-introduction.html) of the series is out.

-------------------

P.S. This is an interesting video to watch on [FP vs OOP](http://www.youtube.com/watch?v=q0BQMbwzPJw).
