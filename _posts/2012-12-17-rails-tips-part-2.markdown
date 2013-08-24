---
layout: post
title: "Rails Tips - part 2"
date: 2012-12-17 20:34
---

Finally found some time and -more importantly- the mood for another post. So here are some more Rails tips I wish I'd known earlier.

## Tip #4: Silence Postgres Warnings

Are you annoyed by messages like these in your Terminal when you run your migrations?

{% highlight sql %}
NOTICE:  CREATE TABLE will create implicit sequence "notification_settings_id_seq" for serial column "notification_settings.id"
NOTICE:  CREATE TABLE / PRIMARY KEY will create implicit index "notification_settings_pkey" for table "notification_settings"
{% endhighlight %}

Well these are notifications, rather than warnings, to inform us about the awesome stuff Postgres is doing for us. But what do they actually tell us?

Postgres has a numeric type called [`serial`](http://www.postgresql.org/docs/current/static/datatype-numeric.html), which is similar to MySQL's `AUTO_INCREMENT`. A `serial` column stores 4-byte integers combined with a sequence to automatically provide auto-incrementing values. That's what the first message is about: Postgres is automatically creating a sequence to make the `id` column -that ActiveRecord will use- function.

On to the second message. This is what the official documentation says: 
> PostgreSQL automatically creates an index for each unique constraint and primary key constraint to enforce uniqueness. Thus, it is not necessary to create an index explicitly for primary key columns.

Pretty clear, Postgres is creating indexes for us, so it also informs us about that.

To the point: We can silence these notifications by adding this simple line under the environments we want, inside `database.yml`:

{% highlight ruby %}
min_messages: WARNING
{% endhighlight %}

Fortunately, we [won't have to do this](https://github.com/rails/rails/commit/052e415f22b98bb45a5245ac8f7aa23c6f32e478) anymore in Rails 4!

## Tip #5: Rails Application Templates

Many times I catch myself going through the same setup steps when starting new Rails apps: writing the same Gemfiles, same Rspec configs, .gitignores, rake tasks, YMLs etc. Wouldn't it be sweet if we could save time by avoiding all this repetition?

Fortunately, Rails application templates lets us do just that: create templates that we can use when we create new apps. There's the relatively unknown (still I wonder why) a DSL that we can use to build our own templates. Here's one of mine for example:

{% highlight ruby %} 
# Fully-armed application template
#
# Postgres, Thin, RSpec, Shoulda, Capybara, FactoryGirl (plus some tricks)
# https://github.com/Agis-/rails_templates/blob/master/full.rb

database_name = ask("What do you want to call the database?")

run "rm public/index.html"
run "mv README.rdoc README.md"

gem "pg"
gem "thin"

require_relative 'testing'

# Database configuration
run "cp -f #{File.expand_path File.dirname(__FILE__)}/lib/database.yml config/"
run "perl -pi -w -e 's/db_name/#{database_name}/g;' config/database.yml"
run "createdb #{database_name}"
run "createdb #{database_name}_dev"
run "createdb #{database_name}_test"
rake "db:create"

# Git
git :init
run "echo *.DS_Store >> .gitignore"
git add: "."
git commit: "-a -m 'Initial commit (full template used)'"
{% endhighlight %}

Pretty self-explanatory, isn't it? The way you use it is with the Rails Generator:

```ruby
rails new myapplication -m a_template.rb
```

Actually I've created my own repo where I collect my own templates: (https://github.com/Agis-/rails_templates). If you find yourself in the same position I suggest you do the same.

There's also the [documentation](http://edgeguides.rubyonrails.org/rails_application_templates.html) in the Edge Guides.

## Tip #6: Quick Benchmarking with Benchmark.ms

As it usually happens with Rails, there's a very convenient way to measure the performance of small bits of code, straight in the Rails console. So open up a console and try it:

{% highlight ruby %}
Benchmark.ms { User.all } #=> 89.33299827575684
{% endhighlight %}

You get the response time in miliseconds. This comes from [`ActiveSupport::Benchmarkable`](https://github.com/rails/rails/blob/master/activesupport/lib/active_support/benchmarkable.rb) along the rest of the benchmarking tools. Nice.

Well, that's all.

PS. In case you missed it, here's the [first part](/2012/05/24/my-rails-notes.html) of this post.
