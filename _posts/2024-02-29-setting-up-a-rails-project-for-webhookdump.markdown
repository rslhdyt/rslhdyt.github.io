---
tags: ["webhookdump", "journey", "rails"]
date: 2024-02-29 00:00:00 +0700
description: The journey has begun!  Let’s set up a new rails project for the Webhookdump application.
published: true
collection: posts
category: blog
id: 84f69ffc-3827-4d0b-864f-c0e67686a7aa
title: Setting Up a Rails Project for Webhookdump
created_time: 2024-03-23T15:26:00.000Z
cover: 
icon: 
last_edited_time: 2024-03-23T23:44:00.000Z
archived: false
created_by_object: user
created_by_id: 6b2c6a42-5dc5-4108-b726-4c02437b814d
last_edited_by_object: user
last_edited_by_id: 6b2c6a42-5dc5-4108-b726-4c02437b814d
---

The adventure begins! I embarked on a journey to set up a new Rails project for an application—Webhookdump.

Before diving into the details, I had to ensure that my local environment was equipped with all the necessary dependencies. For the record, I won't be diving into the details of how I installed and set up these dependencies.

Dependencies:

1. Ruby
2. Rails gem
3. SQLite3

With the prerequisites in place, the first step on my journey was to create the Rails application for Webhookdump:

```bash
$ rails new webhookdump
      create
      create  README.md
      create  Rakefile
      create  .ruby-version
      create  config.ru
      create  .gitignore
      create  .gitattributes
      ...
      
Bundle complete! 15 Gemfile dependencies, 83 gems now installed.
Use `bundle info [gemname]` to see where a bundled gem is installed.
```

Once the project was set up, it was time to configure the database. The database configuration is located in `config/database.yml`. By default, Rails uses SQLite as its database. I decided to modify the database name to match the project name.

```yaml
# SQLite. Versions 3.8.0 and up are supported.
#   gem install sqlite3
#
#   Ensure the SQLite 3 gem is defined in your Gemfile
#   gem "sqlite3"
#
default: &default
  adapter: sqlite3
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
  timeout: 5000

development:
  <<: *default
  database: storage/webhookdump_development.sqlite3

# Warning: The database defined as "test" will be erased and
# re-generated from your development database when you run "rake".
# Do not set this db to the same as development or production.
test:
  <<: *default
  database: storage/webhookdump_test.sqlite3

production:
  <<: *default
  database: storage/webhookdump_production.sqlite3

```

Then came the moment of truth. I started the web server by running `rails server`.

```bash
$ rails server

=> Booting Puma
=> Rails 7.1.3.2 application starting in development
=> Run `bin/rails server --help` for more startup options
Puma starting in single mode...
* Puma version: 6.4.2 (ruby 3.3.0-p0) ("The Eagle of Durango")
*  Min threads: 5
*  Max threads: 5
*  Environment: development
*          PID: 38945
* Listening on http://127.0.0.1:3000
* Listening on http://[::1]:3000
Use Ctrl-C to stop
```

Next, I opened my browser and navigated to `127.0.0.1:3000`. And voila!

![](/assets/images/posts/2e86ea52-4819-42c8-8138-95751dd82deb-Untitled.png)

And that was it - the Rails application, Webhookdump, was now installed on my local machine. In the next chapter of this journey, I'll share how I began to add features to the application. Stay tuned!

<br />


