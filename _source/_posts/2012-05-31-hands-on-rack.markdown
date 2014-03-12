---
layout: post
title: "Hands-on Rack"
date: 2012-05-31 18:06
---

> Rack provides a minimal interface between webservers supporting Ruby and Ruby frameworks.

Our first project in the [Code Reading](https://tinyletter.com/codereading) group was Rack. At first I thought it was going to be a complex project to go through but Rack's logic is really simple. This is an overview of how it actually works.

Indeed, the Rack API is as simple as it gets:

> A Rack application is any Ruby object that responds to `call`. It takes exactly one argument, the environment hash, and returns an Array of exactly three values: The status, the headers, and the body.

Based on this, a simple Rack app could be this:

{% highlight ruby %}
class MyApp
  def call(env)
    [200, {"Content-Type" => "text/plain"}, ["Hello world!"]]
  end
end

run MyApp
{% endhighlight %}

We can run the app by saving this class to a file named `config.ru` and use the `rackup` tool that Rack provides for easily running Rack apps. `#run` and `#use` are two of the methods that the [Rack DSL](https://github.com/rack/rack/blob/master/lib/rack/builder.rb) provides:

{% highlight bash %}
$ rackup config.ru
{% endhighlight %}

This will boot Thin (or WEBrick if you don't have Thin installed). You can override the server if you want. Point to http://0.0.0.0:9292 to see your app.

It gets simpler! Lambdas also respond to `call` (since they're Proc objects), so in our rackup file we could just do:

{% highlight ruby %}
run lambda { |env| [200, { "Content-Type => "text/plain" }, ["OK"]] }
{% endhighlight %}

[builder.rb](https://github.com/rack/rack/blob/master/lib/rack/builder.rb) implements the simple DSL we can use to add Middlewares to the Stack. Now we can use `use` method to add Middlewares to the stack.

{% highlight ruby %}
class MyMiddleware
  def initialize(app)
    @app = app
  end

  def call(env)
    env["some_header"] = "I'm a Middleware"
    @app.call(env)
  end
end

use MyMiddleware
run lambda { |env| [200, { "Content-Type" => "text/plain" }, ["OK", env["some_header"]]] }
{% endhighlight %}

We can see in our browser that this adds the headers. The important thing here is to understand how the Middleware stack works. In our case, the Proc is the endpoint -our main application- and above it sits our new Middleware.

If we take a look at `builder.rb` (which is a key file to understanding the stack) we can see that `#use` along with `#to_app` builds an array of Procs that each take an app and builds another app out of it, by prepending a Middleware to it (a Rack app with a Middleware in front of it is a Rack app). Note Lines 25 & Line 31 which do the trick:

{% highlight ruby %}
# Specifies middleware to use in a stack.
#
#   class Middleware
#     def initialize(app)
#       @app = app
#     end
#
#     def call(env)
#       env["rack.some_header"] = "setting an example"
#       @app.call(env)
#     end
#   end
#
#   use Middleware
#   run lambda { |env| [200, { "Content-Type => "text/plain" }, ["OK"]] }
#
# All requests through to this application will first be processed by the middleware class.
# The +call+ method in this example sets an additional environment key which then can be
# referenced in the application if required.
def use(middleware, *args, &block)
  if @map
    mapping, @map = @map, nil
    @use << proc { |app| generate_map app, mapping }
  end
  @use << proc { |app| middleware.new(app, *args, &block) }
end

def to_app
  app = @map ? generate_map(@run, @map) : @run
  fail "missing run or map statement" unless app
  @use.reverse.inject(app) { |a,e| e[a] }
end
{% endhighlight %}

So if we had an imaginary stack like this

{% highlight ruby %}
# top of the stack
use CoolMiddleware
use AnotherMiddleware
use YoMiddleware
run OurApp # the endpoint, our main app
# bottom of the stack
{% endhighlight %}

then the server in response to a request from a client, would go through the stack in reverse order. The first Middleware, "YoMiddleware" gets initialized with "OurApp", thus creating a Rack app. Then "AnotherMiddleware" gets initialized with that Rack app that was just created and the same process is repeated for next Middlewares till the stack is finished.

It could look something like this:

{% highlight ruby %}
yo_middleware = YoMiddleware.new(OurApp) # iteration 1
another_middleware = AnotherMiddleware.new(yo_middleware) # iteration 2
cool_middleware = CoolMiddleware.new(another_middleware) # iteration 3
{% endhighlight %}

Each Middleware in the stack knows the next one but nothing else. Control is passed by calling the next Middleware. Middlewares can run before or after an application, it depends on where the Middleware wants to work.

The real power comes when you add, remove, move Middlewares to the stack at will and the remaining Middlewares can not care less. The implementation is so simple yet clever that an endpoint looks exactly the same as an endpoint with a middleware in front of it.

Our next project in Code Reading will propably be Sinatra. Stay tuned.

### More resources
* [Rack Homepage](http://rack.github.com/)
* [A great introduction to Rack](http://ruby.about.com/od/rack/a/What-Is-Rack.htm)
* [Ryan Bates on Rack Middleware](http://railscasts.com/episodes/151-rack-middleware)
* [Rack Summary by @tony_kaira (code reader)](https://gist.github.com/2731566)
