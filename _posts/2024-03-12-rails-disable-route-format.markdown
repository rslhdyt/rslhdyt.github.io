---
tags: ["rails", "webhookdump", "security"]
date: 2024-03-12 00:00:00 +0700
description: Learn how to disable Rails route formats (.xml, .json) to prevent unwanted format requests and clean up your error logs. Simple solutions to improve your Rails app's security and monitoring.
published: true
collection: posts
category: blog
id: ed4830ea-983f-49b1-924f-ddbebd12c67c
title: Rails Disable Route Format
created_time: 2024-03-13T00:57:00.000Z
cover: 
icon: 
last_edited_time: 2024-10-24T08:01:00.000Z
archived: false
created_by_object: user
created_by_id: 6b2c6a42-5dc5-4108-b726-4c02437b814d
last_edited_by_object: user
last_edited_by_id: 6b2c6a42-5dc5-4108-b726-4c02437b814d
---

Today, while checking my APM (Application Performance Monitoring) tools, I found some strange errors in the reports. After looking closer, I realized these errors were coming from people trying to access my webhook URLs by adding format extensions like `/blog.xml` or `/blog.json`. Even though these endpoints don't exist in my app, these requests were creating unnecessary error alerts.

In this post, I'll show you how to disable these format extensions in Rails to avoid similar issues.

## What's Happening?

By default, Rails allows URLs to end with format extensions (like .xml, .json, .csv). This means when someone tries to access:

```text
GET /webhooks.xml
GET /webhooks.json
```

Rails will try to respond to these requests. If you haven't set up your app to handle these formats, it returns a 404 error, which then shows up in your error monitoring tools.

## How to Fix This

There are three ways to solve this problem. Choose the one that fits your needs best:

### Option 1: Turn Off Formats for All Routes

This is the simplest solution. In your `config/routes.rb` file, add:

```ruby
Rails.application.routes.draw do
  default_format :html

  # Your routes here
  resources :webhooks, format: false
end
```

### Option 2: Turn Off Formats for Specific Routes Only

If you want to keep formats for some routes (like an API) but disable them for others:

```ruby
Rails.application.routes.draw do
  # API routes that can still use formats
  namespace :api do
    resources :webhooks
  end

  # Regular routes without formats
  scope format: false do
    resources :webhooks
  end
end
```

### Option 3: Use Format Constraints

For more control over which routes accept formats:

```ruby
Rails.application.routes.draw do
  constraints format: false do
    get '/blog', to: 'blog#index'
    resources :posts
  end

  # Routes outside this block can still use formats
end
```

source constraint: [https://guides.rubyonrails.org/routing.html#specifying-constraints](https://guides.rubyonrails.org/routing.html#specifying-constraints)

## What to Check After Making Changes

1. Run `rails routes` to see how your changes affected your routes
2. Watch your logs to make sure real requests aren't being blocked
3. Test your changes in development before pushing to production

## Results From My Experience

After I made these changes:

- My APM stopped showing errors from these format requests
- My error logs became cleaner and easier to read
- It became easier to spot real issues in the logs


