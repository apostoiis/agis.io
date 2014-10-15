---
layout: post
title: "Dabbling in Erlang, part 2: A minimal introduction"
date: 2013-10-12 15:34
---

It's been a while but in the [last post](http://agis.io/2013/05/19/dabbling-in-erlang-hello-function.html) we had a short introduction into functional programming in general. Let's actually dive into the basic concepts of the language.

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

But this error is not exactly what we expected. In Erlang, the `=` operator does actually pattern matching (there are no LHS/RHS values). A pattern match is done like so:

{% highlight erlang %}
Pattern = Expression
{% endhighlight %}

The `Pattern` consists of data structures that may contain both bound and unbound variables. The `Expression` part may contain data structures, bound variables, mathematical operations and function calls. If the pattern match succeeds, any unbound variables will be bound and the value of the expression will be returned. Using pattern matching we can extract data out of complex structures:

{% highlight erlang %}
> {_, Name, Surname} = {person, "John", "Doe"}.
{person,"John","Doe"}
> Name.
"John"
> Surname.
"Doe"
> {_, Name, "bla"} = {person, "John", "Doe"}.
** exception error: no match of right hand side value {person,"John","Doe"}
{% endhighlight %}

We extract the name and surname and bound them to the `Name` and `Surname` variables. The second pattern match fails because *"bla"* does not equal to *"Doe"*.

Note that all variable names must begin with a capital letter.

But this isn't the end. Using pattern matching we can control the execution flow of our programs and write more conscise code. Let's say we want a function which, depending on the first argument we pass it, will print something to the screen.

Here's how we might do this in Ruby:

{% highlight ruby %}
def greet(inc)
  case inc
  when "Hello"   then "Hi"
  when "Goodbye" then "See you"
  else inc
  end
end

greet("Hello")   # => "Hi"
greet("Goodbye") # => "See you"
greet("blabla")  # => "blabla"
{% endhighlight %}

Here's a direct port to Erlang:

{% highlight erlang %}
greet(Inc) ->
  case Inc of
    "Hello"   -> "Hi";
    "Goodbye" -> "See you";
    _         -> Inc
  end.
{% endhighlight %}

We essentially use the `case` syntax which does pattern matching. But that's not idiomatic Erlang; we can do better: pattern match in the head of the function clause (the part before the `->` is called the *head* and after it comes the *body*).

{% highlight erlang %}
greet("Hello")   -> "Hi";
greet("See you") -> "See you";
greet(Other)     -> Other.
{% endhighlight %}

A function is defined as a collection of clauses. Each clause specifies the expected argument patterns and a body of expressions to be evaluated. `greet/1` is still one function but without the use of `case` or `if` statements. The `name/arity` notation is used to describe functions. So `greet/1` means the `greet` function which accepts *one* argument. It's worth mentioning that when the compiler creates a binary search tree with the `greet` clauses, so pattern matching is very efficient.

But it gets better.

Let's say you want to calculate the area of a shape which could be either a *square* or a *circle*.

In Ruby we could define two different functions:

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

and we would use it like this:

{% highlight erlang %}
> area({square, 15}).
225
> area({circle, 5}).
78.5
> area({wtf, 5}).
** exception error: no function clause matching area({wtf,5})
{% endhighlight %}

In the Erlang version, we're defining one function with multiple clauses. We also don't have to explicitly raise an error as we did in the Ruby version, since pattern matching will do this for us.

Note that we're using a composite data type called a *tuple* (that thing inside the curly brackets). I won't get into details here, but you will see tuples everywhere in Erlang.

It's important to point out that a function is identified by its name and its arity. Two functions with the same name but different arity are two completely different functions that just happen to share the same name. Think of it like two people with the same first name, but different last names (ie. arity).

This means we can do this:

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

You can notice that we terminate each definition with a period, which means we are essentially defining a different function every time. `_`, like in Ruby, denotes that we don't care about the actual values of the arguments.

## Guards
*Guards* can be thought of as additional constraints that can be applied when pattern matching.

Let's define a function that tells us if the given number is even or odd:

{% highlight erlang %}
is_what(X) when X rem 2 == 0 -> even;
is_what(X) when X rem 2 /= 0 -> odd.
{% endhighlight %}

Notice the `when <guard>` part? We can use our new function like this:

{% highlight erlang %}
is_what(3). % => odd
is_what(2). % => even
{% endhighlight %}

To achieve the same result in Ruby, we would do something like:

{% highlight ruby %}
def is_what(x)
  return :even if x % 2 == 0
  :odd
end
{% endhighlight %}

The cool thing is that guards can also be used in function heads, `case` and `if` statements.

Essentially they provide us with a succinct way to describe what we want out of a list.

## Lists
Before we can do anything cool, we must really meet the most used data structure in Erlang, the *List*.

*Lists* are the bread & butter of many functional programming languages and are very efficient data types.

The first element of the list is called the *head* and the rest of the list is called the *tail*. Using pattern matching and the cons operator (`|`) we can easily extract those two out of a list:

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

To illustrate the recursive definition of lists, it helps to understand that all of
the following notations:

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
Functions in Erlang are first-class citizens. This means we can manipulate them like any other data, passing them as arguments to other functions or making them the return values of other functions.

Let's implement our own [`map/2`](http://en.wikipedia.org/wiki/Map_(higher-order_function)) function. Turns out it's very easy to do using lists and recursion:

{% highlight erlang %}
map(F, [])    -> [];
map(F, [H|T]) -> [F(H) | map(F, T)].
{% endhighlight %}

Our `map` function accepts another function as it's first argument and applies that function to every element in the list (ie. the second argument). It's a [high-order function](http://en.wikipedia.org/wiki/Higher-order_function).

The first definition is our *base case* which, when reached, the function will stop recursing and return the result. Applying a function to an empty list should return an empty list.

But what function should we pass to it? We can pass it a `fun` which will double the passed argument by 2. You can think of `fun`s as anonymous functions, something like blocks in Ruby (well, not quite):

{% highlight erlang %}
> map(fun(X) -> X*2 end, [1,2,3]).
[2,4,6]
{% endhighlight %}

That's awesome! In two lines of code we created our own `map` function.

Let's try passing our `is_what/1` function that we defined earlier:

{% highlight erlang %}
> F = fun(X) -> is_what(X) end.
#Fun<erl_eval.6.17052888>
> map(F, [1,2,3]).
[odd,even,odd]
{% endhighlight %}

Neat! Functions are first-class citizens in Erlang and we can manipulate them the same way we manipulate all other data. We can even define a function wich returns another function:

{% highlight erlang %}
times(X) ->
  fun(Y) -> X*Y end.
{% endhighlight %}

We can now reuse our `times/1` function:

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

Another common task is to *filter* out elements that meet a particular
requirement, for example even numbers:

{% highlight erlang %}
evens([]) ->
  [];
evens([H|T]) ->
  case H rem 2 == 0 of
    true -> [H | evens(T)];
    _    -> evens(T)
  end.
{% endhighlight %}

Let's see it in action:

{% highlight erlang %}
> hi:evens([1,2,3,4,5]).
[2,4]
{% endhighlight %}

Note that the order of definitions is important: each definition is evaluated from top to bottom, so if we wanted to add a clause to report back whatever invalid shape was received, we might do: 

{% highlight erlang %}
area(Other)            -> {unknown, Other};
area({circle, Radius}) -> 3.14 * Radius * Radius;
area({square, Side})   -> Side * Side.
{% endhighlight %}

But this would not work as expected, since the first clause would always match whatever argument we passed to `area/1`.
The following demonstrates:

{% highlight erlang %}
> area({shapeFromSpace, 5}).
{unknown,{shapeFromSpace,5}}
> area({circle, 5}).
{unknown,{circle,5}}
> area({rectangle, 5}).
{unknown,{rectangle,5}}
{% endhighlight %}

To fix this we have to move the first clause to the bottom, in order to match only unknown shapes:

{% highlight erlang %}
area({circle, Radius}) -> 3.14 * Radius * Radius;
area({square, Side})   -> Side * Side;
area(Other)            -> {unknown, Other}.
{% endhighlight %}

## List comprehensions
It's a very common pattern to `map` and `filter` lists. In other words to apply a function to every element of a list and from the resulting list, select those elements that meet a given requirement. *List comprehensions* provides us with a notation to achieve this result in a succinct yet powerful way.

Let's say we want to pick all *even* numbers in a list and multiply them by *2*. List comprehensions to the rescue:

{% highlight erlang %}
> [X*2 || X <- [1,2,3,4,5], X rem 2 == 0].
[4,8]
{% endhighlight %}

*Lists comprehensions* are essentially tools to build and modify lists and are based on the [set-builder notation](http://en.wikipedia.org/wiki/Set-builder_notation).

That's a fairly simple example. Let's try something slightly more advanced by using multiple generators (ie. the part after the `||`):

{% highlight erlang %}
> [{X,Y} || X <- lists:seq(1,4), X rem 2 == 0, Y <- lists:seq(X,4)].
[{2,2},{2,3},{2,4},{4,4}]
{% endhighlight %}

Here the expression is the tuple `{X, Y}`, then follows the generator `X <- lists:seq(1,4)` which essentially means *"`X` comes from `lists:seq(1,4)`"* ([`lists:seq/2`](http://www.erlang.org/doc/man/lists.html#seq-2) is a built-in function that generates a sequence of integers), then follows the guard `X rem 2 == 0` and finally another generator.

As we can see, list comprehensions result in more succinct and idiomatic code.

## That's not all folks
In this short journey, we ran into *lists*, *pattern matching*, *guards*, *functions* and *list comprehensions*. What we learned can be summarized as follows:

* *List* is the most useful data structure in Erlang. Common operations on them are very efficient: extracting the *head* or *tail* of a list and inserting an element at the beginning of the list are constant time operations. The cons operator is the basic tool for manipulating lists.

* *Pattern matching* is a fundamendal characteristic of the language and a very powerful concept that is everywhere in Erlang programs. It results in [more readable code](http://existentialtype.wordpress.com/2011/03/15/boolean-blindness/) (vs `if-else` statements). *Guards* are a great tool that provide additional power to pattern matching.

* *Functions* are first-class citizens and are treated like any other data type. You can pass them as arguments, assign them to variables and use them as the return values of other functions.

This was a minimal introduction to some of the basic concepts of Erlang. The next post will be about the concurrency features of the language.

[Discuss this post on HackerNews.](https://news.ycombinator.com/item?id=6538888)
