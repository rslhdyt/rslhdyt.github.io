---
tags: ["rails", "email", "testing"]
date: 2023-09-06
description: Previewing your mailer class in rails
published: true
category: blog
id: a26783a3-ea2a-455c-8d57-2c23c7dbcc35
title: Rails Email Preview
created_time: 2022-10-26T07:13:00+00:00
cover: 
icon: 
last_edited_time: 2023-11-21T00:32:00+00:00
archived: false
---

When working with features that need to send an email, sometimes we find out that testing the email content is a little bit painful. Usually, we will call the function that sends the email and inspect the email content and view it through a development mail server such as mailcatcher or mailtrap. Most of the time we don’t want to go through all processes to only see the email body.

Luckily, Rails has a feature to preview your email mailer without having to use the email server at all. In this tutorial, I will explain how to set up a mailer preview in the Rails framework.

Let’s assume we already have a mailer class like this one.

```ruby
class UserMailer < ApplicationMailer
  def welcome(user)
    @user = user
    mail(to: user.email, subject: 'Welcome!')
  end
end
```

```ruby
Hi <%= @user.full_name %>
<br><br>
Welcome to the club!
```

<br />

In order to enable the email preview for user mailer, we need to create user mailer preview class in `test/mailers/previews` directory. 

> 💡 if you are using rspec as testing framework, usually the directory is inside the spec folder `spec/mailers/previews`.

```ruby
class UserMailerPreview < ActionMailer::Preview
  def welcome
    user = User.first

    UserMailer.welcome(user)
  end
end
```

Then, you can check the email preview by accessing this url in your local url development. `[http://localhost:3000/rails/mailer](http://localhost:3000/rails/mailer)` . Make sure your local development server are running.

If everything goes right, you will see the list available email preview like this.

![](https://prod-files-secure.s3.us-west-2.amazonaws.com/14f037b2-f507-4c22-9546-eba261dcbb3e/7f7cdfba-29b7-45bc-8e98-dde02f4864bd/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45HZZMZUHI%2F20231121%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20231121T030405Z&X-Amz-Expires=3600&X-Amz-Signature=753f2be2009ff0b6f24705a6f443aeb91477993e95cd6d6bdb7f996bac373ab9&X-Amz-SignedHeaders=host&x-id=GetObject)

And if you click the welcome email preview it will show the preview of user mailer welcome.

![](https://prod-files-secure.s3.us-west-2.amazonaws.com/14f037b2-f507-4c22-9546-eba261dcbb3e/745e2fef-1c12-4aa5-93ef-5ed5cdb0d837/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45HZZMZUHI%2F20231121%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20231121T030405Z&X-Amz-Expires=3600&X-Amz-Signature=dc858079d962df2bfd7c01e1ae3b6d06ba9b2791e66cab4d3f568991386e36be&X-Amz-SignedHeaders=host&x-id=GetObject)

Voila, that’s all. I hope this tutorial will make you more productive when working with a feature with an email.

---

sources:

- [https://guides.rubyonrails.org/action_mailer_basics.html#previewing-emails](https://guides.rubyonrails.org/action_mailer_basics.html#previewing-emails)
