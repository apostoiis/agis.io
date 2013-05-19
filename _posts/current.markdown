---
layout: post
title: "Dabbling in Erlang #1: Hello functional"
date: 
---

Recently I've been dabbling in Erlang and I'm enjoying it a lot. I decided to start a series of posts in which I'll address things that I learned until now.

## Meet Erlang
[Erlang](http://en.wikipedia.org/wiki/Erlang_(programming_language)) is a functional, concurrent, dynamically-typed language built at Ericsson around 1986 in order to power its telecom infrastructure. 

Erlang shines when you want high-reliable, distributed and scalable system (surprise, exactly what telecom applications need). That's because of the fact that it's a functional language, has a concurrency model that proves to be a great fit, and solid error handling capabilities.

But before I dive into the language, I'd like to talk a little about functional programming in general. In fact this will be the subject of this first post of the series.

## How I stopped worrying about state and loved the function
Coming to a declarative language like Erlang from an imperative language like Ruby, certainly requires a mental switch:

You have to stop thinking in terms of mutating the program's state and start thinking about your functions and the data they 'll return. You start describing what you want to achieve rather than how you will achieve it. You can no longer have your lovely side-effects. There are no things like `a += 1` here.

Where there used to be `for` loops, there is `recursion` instead.

This:
{% highlight ruby %}
sum = 0

for i in [1, 2, 3, 4]
  sum += i
end

sum
{% endhighlight %}

becomes:

{% highlight erlang %}
sum([H|T]) -> H + sum(T);
sum([])    -> 0.

sum([1, 2, 3, 4]).
{% end %}

See? In the second version there's no mutation at all, we just define a recursive function which gets in some data and spits out some other data, without messing with the outside world at all. But what benefits do this lack of side-effects grants us?

### Easier to reason about
Since there are no side-effects, it tends to be easier to reason about functional code. To understand what a function does, you only have to look its code and nothing else. This, most times is not true with imperative programs, where in order to understand what a function does, you may have to look and understand a lot of different pieces in the code and understand all the external dependencies. In functional code there are no external dependencies.

### Easier debugging
This in turn, leads to programs which are easier to debug, since you can immediately trace the source of bugs that occur. Add single assignment to the mix and debugging becomes a lot simpler.

### Easier shift to multicore
It's well known that parallelizing code where there is shared mutable state is hard. You have to mess with locks and mutexes and then bad things will eventually happen: deadlocks, race-conditions etc. Having immutable state and no shared memory has enormous advantages when you want to make your program run in parallel. All you have to do make your code concurrent -something that Erlang is built for- and the shift to multicore is almost transparent.

### Easier testing
Since functions are deterministic and the result totally depends on the input and nothing else, it means that if they produce the same answer today, they will produce the same answer tomorrow.

## So why hasn't functional programming taken over yet?
If there are all these advantages, why we see a lot of C, Java, Ruby, Python and not so much Haskell, F#, Clojure, Erlang etc.? 

Well, first of all, functional programming also lives in imperative languages. When you're doing a `.map` or `.inject` or using blocks in Ruby, you're programming in a functional manner. OK, since we got that out of the way, we can answer to the point:

First of all, side-effects can be as much a disadvantage as it can be an advantage. It all depends in the problem we want to solve. Truth is, in most real-world problems you want to have side-effects: when a user of your web application clicks on a button to make the purchase, it just makes more sense to alter his current balance state by doing something like `balance -= 50` that to construct a new balance with the new state. You also want to persist his updated balance into the database.

Also, as I've said before, thinking in a functional manner is a difficult skill and requires a big mental shift. This is propably  mostly because when we learn about programming we're taught to think in an imperative way. Imperative languages are dominating the industries since ever.

## So what now?
You may have read it a thousand times but it really is the most reasonable thing to say: There is no silver-bullet. It all depends on the problem you're facing. I'd choose Erlang or another functional language in a problem where concurrency would be important. I'd choose Ruby or an object-oriented language in the backend of a web application.

I believe that multi-paradigm is the way of the future (is it not the way of the present really?). More and more we see big companies and start-ups to use different technologies, combined to provide the best solutions to the problems they face.

After all, I think learning functional programming is a great challenge to take. It really trains areas of the brain you may have forgotten about, and will make you a better programmer for sure.
