---
tags: ["rails", "email", "testing"]
date: 2023-09-06 00:00:00 +0700
description: Previewing your mailer class in rails
published: true
category: blog
id: a26783a3-ea2a-455c-8d57-2c23c7dbcc35
title: Rails Email Preview
created_time: 2022-10-26T07:13:00.000Z
cover: 
icon: 
last_edited_time: 2023-12-08T03:08:00.000Z
archived: false
created_by_object: user
created_by_id: 6b2c6a42-5dc5-4108-b726-4c02437b814d
last_edited_by_object: user
last_edited_by_id: 6b2c6a42-5dc5-4108-b726-4c02437b814d
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

![](/assets/images/posts/Untitled.png)

<em></em>

And if you click the welcome email preview it will show the preview of user mailer welcome.

![](/assets/images/posts/Untitled.png)

<em></em>

Voila, that’s all. I hope this tutorial will make you more productive when working with a feature with an email.

---

sources:

- [https://guides.rubyonrails.org/action_mailer_basics.html#previewing-emails](https://guides.rubyonrails.org/action_mailer_basics.html#previewing-emails)


