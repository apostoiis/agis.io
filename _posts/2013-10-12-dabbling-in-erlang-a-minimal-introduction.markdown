---
layout: post
title: "Dabbling in Erlang, part 2: A minimal introduction"
date: 2013-10-12 15:34
---

It's been a while but in the [last post](http://agis.kontext.gr/2013/05/19/dabbling-in-erlang-hello-function.html) we had a short introduction into functional programming in general. Let's actually dive into the basic concepts of the language.

## Single assignment & pattern matching
In Erlang we should think variables in a mathematical sense of the term, which means that *once you've bound a variable, you cannot change its value* . If you state that `X` is `5`, then it will always be `5` (a variable is called *bound* if already contains a value, *unbound* otherwise):

{% highlight erlang %}
Eshell V5.10.1  (abort with ^G)
> X = 5.
5
> X = 6.
** exception error: no match of right hand side value 6
> X = 5.
5
{% endhighlight %}

But this error is not exactly what we expected. In Erlang, the `=` operator does actually pattern matching (there are no LHS/RHS  values). A pattern match is written like this:

{% highlight erlang %}
Pattern = Expression
{% endhighlight %}

The `Pattern` consists of data structures that may contain both bound and unbound variables. The `Expression` part may contain data structures, bound variables, mathematical operations and function calls. If the pattern match succeeds, any unbound variables will be bound and the value of the expression will be returned. Using pattern matching we can extract data of complex data structures:

{% highlight erlang %}
> {_, Name, Surname} = {person, "Takis", "Doe"}.
{person,"Takis","Doe"}
> Name.
"Takis"
> Surname.
"Doe"
> {_, Name, "bla"} = {person, "Takis", "Doe"}.
** exception error: no match of right hand side value {person,"Takis","Doe"}
{% endhighlight %}

We extract the name and surname and bound them to the `Name` and `Surname` variables. The second pattern match of course fails because *"bla"* does not equal to *"Doe"*.

Note that all variable names must begin with a capital letter.

But this isn't the end. Using pattern matching we can control the execution flow of our programs and write more conscise yet powerful code. Let's say we want a function which depending on the first parameter we pass it, will print something to the screen, here's how we might do this in Ruby:

{% highlight ruby %}
def greet(inc)
  case inc
  when "Hello"   then "Hi"
  when "Goodbye" then "See you"
  else                inc
  end
end

greet("Hello")   # => "Hi"
greet("Goodbye") # => "See you"
greet("blabla")  # => "blabla"
{% endhighlight %}

Here's a direct translation to Erlang:

{% highlight erlang %}
greet(Inc) ->
  case Inc of
    "Hello"   -> "Hi";
    "Goodbye" -> "See you";
    _         -> Inc
  end.
{% endhighlight %}

We essentially use the `case` syntax which does pattern matching. But that's not the Erlang way, we can do better: pattern match in the head of the function's clause (the part before the `->` is called the *head* and after it comes the *body*).

{% highlight erlang %}
greet("Hello")   -> "Hi";
greet("See you") -> "See you";
greet(Other)     -> Other.
{% endhighlight %}

A functions is defined as a collection of clauses. Each clause specifies the expected argument patterns and a body of expressions to be evaluated. `greet/1` is still one function but without the use of `case` or `if` statements. The `name/arity` notation is used to describe functions. So `greet/1` means the `greet` function which accepts 1 argument. It's worth mentioning that the in the compiler meets the different `greet` clauses, it ends up creating a binary search tree, so pattern matching is very efficient.


But it gets better, let's say you want to calculate the area of a shape which could be either a square or a circle. In Ruby we could define *two* different functions:

{% highlight ruby %}
def area_square(side)
  side * side
end

def area_circle(radius)
  3.14 * radius * radius
end

area_square(15) # => 225
area_circle(5)  # => 78.5
{% endhighlight %}

or using a single function we might do:

{% highlight ruby %}
def area(shape, x)
  if shape == :square
    x * x
  elsif shape == :circle
    3.14 * x * x
  else
    raise "Unknown shape"
  end
end

area(:square, 15) # => 225
area(:circle, 5)  # => 78.5
{% endhighlight %}

In Erlang we could do the following, using pattern matching and one function:

{% highlight erlang %}
area({circle, Radius}) -> 3.14 * Radius * Radius;
area({square, Side})   -> Side * Side.
{% endhighlight %}

and we would use it like this

{% highlight erlang %}
> area({square, 15}).
225
> area({circle, 5}).
78.5
> area({wtf, 5}).
** exception error: no function clause matching area({wtf,5})
{% endhighlight %}

In the Erlang version, we're defining one function with multiple clauses. We also don't have to explicitly raise an error as we do in the Ruby version, the pattern match will fail so an exception will be raised anyway.

Note that we're using a composite data type called *tuple* (that thing inside the curly brackets). I won't get into details but you will see tuples everywhere in Erlang.

It's important to point out that a function is identified by its name and its arity. Two functions with the same name but different arity are two completely different functions that just happen to share the same name. This means could do this:

{% highlight erlang %}
adder(_)       -> "I'm someone".
adder(_, _)    -> "I'm another one".
adder(_, _, _) -> "I'm yet another one".
{% endhighlight %}

{% highlight erlang %}
> adder(3,2)
"I'm another one"
> adder(1)
"I'm someone"
> adder(5,6,7)
"I'm yet another one"
{% endhighlight %}

You can notice that we terminate each definition with a full-stop, which means we're essentially defining a different function every time. The `_`  means that we don't care about the value of the argument.

## Guards
Guards are additional constraints that are used in conjuction with pattern matching to provider even more power:

Let's define a function that tells us if the given number is even or odd. Notice the `when <guard>` syntax:

{% highlight erlang %}
is_what(X) when X rem 2 == 0 ->
  even;
is_what(X) when X rem 2 /= 0 ->
  odd.
{% endhighlight %}

We can use it like this:

{% highlight erlang %}
is_what(3). % => odd
is_what(2). % => even
{% endhighlight %}

Whereas in Ruby we would do something like:

{% highlight ruby %}
def is_what(x)
  if x % 2 == 0
  	:even
  else
  	:odd
  end
end
{% endhighlight %}

Guards can be used in function heads, `case` and `if` statements.

## Lists
Before we can do anything cool, we must really meet the most used data structure in Erlang, the *List*. Lists are the bread and butter of many functional programming languages and are very efficient data types.

The first element of a List is called the *head* and the rest of the List is called the *tail*. Using pattern matching and the cons operator `|` we can easily extract those out from a List:

{% highlight erlang %}
> [Head|Tail] = [1,2,3,4,5].
[1,2,3,4,5]
> Head.
1
> Tail.
[2,3,4,5]
> [A|B] = Tail.
[2,3,4,5]
> A.
2
> B.
[3,4,5]
> ["Look what I did!"|B].
["Look what I did!",3,4,5]
{% endhighlight %}

To illustrate the recursive definition of lists, all of the following list notations:

{% highlight erlang %}
[1,2,3,4]
[1,2,3,4|[]]
[1,2|[3,4]]
[1,2|[3,4|[]]]
{% endhighlight %}

are simply syntactic sugar for:
{% highlight erlang %}
[1|[2|[3|[4|[]]]]]
{% endhighlight %}

Note that the head of the list `[1]` is `1` and the tail is itself an empty list (`[]`).

## Functional programming for real
Functions in Erlang are first-class citizens, which means we can manipulate them like any other data, passing them functions or make them the results of other functions.

Let's implement our own [`map`](http://en.wikipedia.org/wiki/Map_(higher-order_function)) function. Turns out it's very easy with using Lists and recursion:

{% highlight erlang %}
map(F, [])    -> [];
map(F, [H|T]) -> [F(H) | map(F, T)].
{% endhighlight %}

Our `map` accepts a function as it's first argument (therefore it's a *high-order function*) and applies that function to every element in the list (ie. the second argument).

The first definition is our *base case* which when hit, the function will stop recursing and return the actual result. Applying a function to an empty list should of course return an empty list.

Note that the order of the definitions is important: had we moved our base case at the bottom, the recursion would never finish because `map(F, [H|T])` would always match (remember? an empty list is also a list with a head and a tail).

But what should we pass to it? We can pass it a `fun` which will double the passed argument by 2. You can think of `fun`s as anonymous functions, something like blocks in Ruby (well, not quite):

{% highlight erlang %}
> map(fun(X) -> X*2 end, [1,2,3]).
[2,4,6]
{% endhighlight %}

Let's try passing our `is_what` function that we defined earlier:

{% highlight erlang %}
> F = fun(X) -> is_what(X) end.
#Fun<erl_eval.6.17052888>
> map(I, [1,2,3]).
[odd,even,odd]
{% endhighlight %}

That's neat! Functions are first-class citizens in Erlang and we can manipulate them the same way we manipulate all other data. We can even define a function wich returns a function:

{% highlight erlang %}
times(X) ->
  fun(Y) -> X*Y end.
{% endhighlight %}

We can now reuse our simple `times` function:

{% highlight erlang %}
> Double = times(2).
#Fun<hi.0.67960283>
> Double(5).
10
> Triple = times(3).
#Fun<hi.0.67960283>
> Triple(3).
9
{% endhighlight %}

Another common task is to filter out elements with a particular property, for example even numbers:

{% highlight erlang %}
evens([]) ->
  [];
evens([H|T]) ->
  case H rem 2 == 0 of
    true -> [H | evens(T)];
    _    -> evens(T)
  end.
{% endhighlight %}

let's see it in action:

{% highlight erlang %}
> hi:evens([1,2,3,4,5]).
[2,4]
{% endhighlight %}

## List comprehensions
It's a very common operation to `map` and `filter` lists. In other words to apply a function to every element of a list and select those elements of a list that have a particular property. *List comprehensions* provides us with a notation to achieve this result in a powerful yet compact way.

The syntax of a list comprehension might be a little confusing at start and requires some [practice](http://trigonakis.com/blog/2011/04/22/introduction-to-erlang-list-comprehension/) to get used to.

Let's say we want to pick all the even numbers in a list and multiply them by two. List comprehensions to the rescue:

{% highlight erlang %}
> [X*2 || X <- [1,2,3,4,5], X rem 2 == 0].
[4,8]
{% endhighlight %}

Lists comprehensions are essentially tools to build and modify lists and are based on the [set-builder notation](http://en.wikipedia.org/wiki/Set-builder_notation).

That's a fairly simple example. Let's try something slightly more advanced by using multiple generators (the part after the `||`):

{% highlight erlang %}
> [{X,Y} || X <- lists:seq(1,4), X rem 2 == 0, Y <- lists:seq(X,4)].
[{2,2},{2,3},{2,4},{4,4}]
{% endhighlight %}

Here the expression is the tuple `{X, Y}`, then follows the generator `X <- lists:seq(1,4)` which essentially means *X comes from lists:seq(1,4)* (`lists:seq/2` is a built-in function that generates a sequence of integers), then follows the guard `X rem 2 == 0` and finally another generator.

So list comprehensions result in more conscise and clear code.

## That's not all folks
We've ran into lists, pattern matching, guards, functions and list comprehensions. What we saw until now can be summed up to this:

* *Lists* are the most useful data structures in Erlang. Common operations on them are very efficient, extracting the head or tail of a list and inserting at the beginning of the list are constant-time. The [cons operator](http://dustin.sallings.org/2010/03/04/erlang-conc.html) is the basic tool for manipulating lists.

* *Pattern matching* is a fundamendal characteristic of the language and a very powerful concept that is everywhere in Erlang programs. It results in [clearer and more readable code](http://existentialtype.wordpress.com/2011/03/15/boolean-blindness/) (vs `if-else` statements). *Guards* are a great tool that provides additional power to pattern matching.

* *Functions* are first-class citizens and are treated like any other data types. You can pass them as arguments, assign them to variables and make them the return values of other functions.

This was a minimal introduction to some of the basic concepts of Erlang. Next post will be about the concurrency features of the language.

[Discuss this post on HackerNews.](https://news.ycombinator.com/item?id=6538888)
