---
layout: post
title: "Rails tips, part 1"
date: 2012-05-24 16:19
---

Mostly like a note to myself, here are some tips that were proved useful to me over time.

Tip #1: Skipping Test::Unit
----------------------------
If you're like me, you prefer RSpec over Test::Unit for its expressiveness and the awesome DSL it provides. Everytime you create a new app, you can use the -T option to skip all the folders and files related to Test::Unit.

{% highlight ruby %}
$ rails new app -T
{% endhighlight %}

Note that this will not only skip the test directory, but also won't create any new test files when you use the built-in controller & model generators.

Tip #2: Console Sandbox mode
---------------------------------------------
Many times I need to insert some data in the database just for testing purposes (eg. to test model validations) and then I want them to vanish immediately.

The sandbox mode lets you do just this: You can create, save & destroy ActiveRecord objects and they'll exist just as long as you terminate the console session.

{% highlight ruby %}
$ rails c --sandbox
{% endhighlight %}

You can see that right before the session gets terminated, Rails automatically rolls back all the transactions made during the session.


Tip #3: Task lists using Rake notes
---------------------------------------
Rake provides you with a great tool you can use to implement your own task lists. Inside .rb and .erb files, you can add notes like this:

{% highlight ruby %}
class Post < ActiveRecord::Base
  # FIXME: whitelist all the attributes that should be whitelisted
  attr_accessible :title, :content, :category

  # TODO: add validations

  # OPTIMIZE: improve the SQL
  scope :published, where(published: true).order("publish_date DESC")
end
{% endhighlight %}

Then in your terminal you can use this command that will scan all of your app's files and list all of your notes and
their exact location.

{% highlight ruby %}
$ rake notes
app/models/post.rb:
  * [2] [FIXME] whitelist all the attributes that should be whitelisted

  * [5] [TODO] add validations

  * [7] [OPTIMIZE] improve the SQL
{% endhighlight %}

As you can see, there are already three built-in types: `TODO`, `FIXME` and `OPTIMIZE`. By default the `notes` task lists all of them. If however you want to see a specific type, you can do it like this:

{% highlight ruby %}
$ rake notes:todo
{% endhighlight %}

However, you're not limited to these three types. You can easily add your own:

{% highlight ruby %}
class PostsController < ApplicationController
  # REMEMBER: you can use any title you want for your task
  def index
  end
end
{% endhighlight %}

Then, to see all the notes of this custom type you could do:

{% highlight ruby %}
$ rake notes:custom ANNOTATION=REMEMBER
  * [2] [REMEMBER] you can use any title you want for your task
{% endhighlight %}

That's all for now. Soon more to come..
