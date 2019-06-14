---
title: "Ruby on Rails"
date: 2019-06-13T16:04:11-06:00
draft: false
weight: 1
tags: ["ruby", "rubyonrails", "rails", "activerecord", "marginalia"]
---

![](/images/activerecord_marginalia-logo.png)

- [Introduction](#introduction)
- [Installation](#installation)
- [Usage](#usage)
- [Fields](#fields)
- [End-to-end example](#end-to-end-example)
    - [Results](#results)
- [References](#references)

### Introduction

[sqlcommenter_rails] adds comments to your SQL statements.

It is powered by [marginalia] and also adds [OpenCensus] information to the
comments if you use the [opencensus gem].

[sqlcommenter_rails] configures [marginalia] and [marginalia-opencensus] to
match the SQLCommenter format.

### Installation

Add this line to your application's Gemfile

```ruby
gem 'sqlcommenter_rails'
```

Then run `bundle` and restart your Rails server.

To enable [OpenCensus] support, add the [`opencensus`][opencensus gem] gem to
your Gemfile, and add the following line in the beginning of
`config/application.rb`:

```ruby
require 'opencensus/trace/integrations/rails'
```

### Usage

All of the SQL queries initiated by the application will now come with comments!

### Fields

The following fields will be added to your SQL statements as comments:

Field | Description | Example
---|---|---
`action` | Controller action name | `index`
`application` | Application name | `MyApp`
`controller` | Controller name | `posts`
`db_driver` | Database adapter class name | `ActiveRecord::ConnectionAdapters::SQLite3Adapter`
`framework` | `rails_v` followed by `Rails::VERSION` | `rails_v6.0.0`
`route` | Request's full path | `/posts`

If the [opencensus gem] is enabled, the following fields will also be added:

Field | Description | Example
---|---|---
`span_id` | OpenCensus Span ID | `3f02d1ace98213ef`
`span_names` | OpenCensus Span names from the current span to root, joined by `~` | `my-span~/posts`
`trace_id` | OpenCensus Trace ID | `d46ccf74712420ce4e62d33eb8260c53`

Note that `controller`, `action`, and `route` fields are only present if the
query was triggered by a web request. For background jobs, a `job` field is
present instead identifying the job.

### End to end example

A Rails 6 [sqlcommenter_rails] demo is available at:
https://github.com/orijtech/sqlcommenter_rails_demo.

The demo is a vanilla Rails API application with [sqlcommenter_rails] and
OpenCensus enabled.

First, we create a vanilla Rails application with the following command:

```shell
gem install rails -v 6.0.0.rc1
rails _6.0.0.rc1_ new sqlcommenter_rails_demo --api
```

Then, we add and implement a basic `Post` model and controller:

```shell
bin/rails g model Post title:text
```

```shell
bin/rails g controller Posts index create
```

Implement the controller:

```ruby
# app/controllers/posts_controller.rb
class PostsController < ApplicationController
  def index
    render json: Post.all
  end

  def create
    title = params[:title].to_s.strip
    head :bad_request if title.empty?
    render json: Post.create!(title: title)
  end
end
```

Configure the routes:

```ruby
# config/routes.rb
Rails.application.routes.draw do
 resources :posts, only: %i[index create]
end
```

Then, we add `sqlcommenter_rails` and OpenCensus:

```ruby
# Gemfile
gem 'opencensus'
gem 'sqlcommenter_rails'
```

```ruby
# config/application.rb
require "opencensus/trace/integrations/rails"
```

Finally, we run `bundle` to install the newly added gems:

```shell
bundle
```

Now, we can start the server:

```shell
bin/rails s
```

In a separate terminal, you can monitor the relevant SQL statements in the server
log with the following command:

```bash
tail -f log/development.log | grep 'Post '
```

#### Results

The demo application has 2 endpoints: `GET /posts` and `POST /posts`.

##### GET /posts

```shell
curl localhost:3000/posts
```

```
Post Load (0.1ms)  SELECT "posts".* FROM "posts" /*
action='index',application='SqlcommenterRailsDemo',controller='posts',
db_driver='ActiveRecord::ConnectionAdapters::SQLite3Adapter',
framework='rails_v6.0.0.rc1',route='/posts',span_id='a52cad0a8d1425ab',
span_names='/posts',trace_id='828f28f7fb3df3dd07ee6478b2016b2a'*/
```

##### POST /posts

```shell
curl -X POST localhost:3000/posts -d 'title=my-post'
```

```
Post Create (0.2ms)  INSERT INTO "posts" ("title", "created_at", "updated_at")
VALUES (?, ?, ?) /*action='create',application='SqlcommenterRailsDemo',
controller='posts',db_driver='ActiveRecord::ConnectionAdapters::SQLite3Adapter',
framework='rails_v6.0.0.rc1',route='/posts',span_id='6ddace73a9debf63',
span_names='/posts',trace_id='ff19308b1f17fedc5864e929bed1f44e'*/
```

### References

| Resource                | URL                                                       |
|-------------------------|-----------------------------------------------------------|
| [sqlcommenter_rails]    | https://github.com/orijtech/sqlcommenter_rails            |
| [marginalia]            | https://github.com/basecamp/marginalia                    |
| [OpenCensus]            | https://opencensus.io/                                    |
| The [opencensus gem]    | https://github.com/census-instrumentation/opencensus-ruby |
| [marginalia-opencensus] | https://github.com/orijtech/marginalia-opencensus         |

[sqlcommenter_rails]: https://github.com/orijtech/sqlcommenter_rails
[marginalia]: https://github.com/basecamp/marginalia
[marginalia-opencensus]: https://github.com/orijtech/marginalia-opencensus
[OpenCensus]: https://opencensus.io/
[opencensus gem]: https://github.com/census-instrumentation/opencensus-ruby
