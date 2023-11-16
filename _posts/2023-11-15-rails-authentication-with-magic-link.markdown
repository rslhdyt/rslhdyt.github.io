---
title: "Rails Authentication: Login with magic link"
layout: post
date: 2023-11-15 22:00
image: /assets/images/markdown.jpg
headerImage: false
tag:
- authentication
- rails 
- tutorial
star: true
category: blog
author: Risal Hidayat
description: Tutorial creating sign in with magic link in Rails
---

![Photo by [Austinchan](https://unsplash.com/@austinchan) / [Unsplash](https://unsplash.com/?utm_source=ghost&utm_medium=referral&utm_campaign=api-credit)](https://images.unsplash.com/photo-1496449903678-68ddcb189a24?ixlib=rb-4.0.3&q=85&fm=jpg&crop=entropy&cs=srgb)

Photo by [Austinchan](https://unsplash.com/@austinchan) / [Unsplash](https://unsplash.com/?utm_source=ghost&utm_medium=referral&utm_campaign=api-credit)

# Introduction

When it comes to letting people into an app, there are different ways to check if they're the real users. One way is to use something called a 'magic link.' This article explains how to make a login system with a magic link in a Rails app. It'll show you how to set it up and why it can be a good idea.

## About magic link

Magic link authentication is a method of logging into a website or application without the need for a password. It relies on the use of a unique URL, also known as a magic link, which is sent to the user's registered email address or phone number. The user can then click on this link to authenticate their identity and gain access to their account.

## How it works?

Typically, the user only needs to enter their email on the authentication page. Next, the application checks whether the email exists. If the email exists, the application sends a signed URL to the user's email. The user then clicks on the link and gets redirected to our application. Finally, the application checks the link's validity before authenticating the user.

## How to implement

To implement the login using magic link you need to do these things:

1. Controller to generate magic link and handle login with link
2. Mailer that will send out the magic link

### Create controller handler

I will name my controller `sign_in_links_controller.rb` and add route like this

```ruby
class SignInLinksController < ApplicationController
	def new
	end

	def create
	end

	def authenticate
	end
end
```

```ruby
get 'sign_in/link', to: 'sign_in_links#new', as: 'new_sign_in_link'
post 'sign_in/link', to: 'sign_in_links#create', as: 'sign_in_links'
get 'sign_in/authenticate/:token', to: 'sign_in_links#authenticate', as: 'authenticate_sign_in_links'
```

### Create login with link view

```ruby
def new
	@user = User.new
end
```

```ruby
<h2>Log in with magic link</h2>

<%= form_with(model: @user, url: sign_in_links_path) do |f| %>
  <div class="field">
    <%= f.label :email %><br />
    <%= f.email_field :email, autofocus: true, autocomplete: "email" %>
  </div>

  <div class="actions">
    <%= f.submit "Send Magic Link" %>
  </div>
<% end %>
```

### Generate magic link

Inside the new controller, you need to create an action to generate the magic link

```ruby
def create
  @user = User.find_by(email: params[:user][:email])

  if @user.present?
    token = @user.signed_id(expires_in: 2.minutes, purpose: :magic_link)

    # send email
    UserMailer.magic_link(@user.id, token)
      .deliver_later

    redirect_to new_user_session_path, notice: 'A temporary login code has been sent to your inbox. Please check it.'
  else
    flash[:notice] = 'Email address not found.'

    render :new, status: :unprocessable_entity
  end
end
```

As you can see above, there are 3 steps to generate the login with magic link.

1. Check the user email existence
2. Generate signed URL token
3. Send the magic link via mailer

Rails 6 introduced `signed_id` method and we can utilize this method to generate authenticate token.

> signed_id method returns a signed id that’s generated using a preconfigured `ActiveSupport::MessageVerifier` instance. This signed id is tamper proof, so it’s safe to send in an email or otherwise share with the outside world. It can furthermore be set to expire (the default is not to expire), and scoped down with a specific purpose. If the expiration date has been exceeded before `find_signed` is called, the id won’t find the designated record. If a purpose is set, this too must match.
> 

[https://api.rubyonrails.org/classes/ActiveRecord/SignedId.html#method-i-signed_id](https://api.rubyonrails.org/classes/ActiveRecord/SignedId.html#method-i-signed_id)

The signed_id method accept 2 optional params

1. expires_in
To set the expiration of the signed ID
2. purpose
To set the name or purpose of signed ID generation

### Login link handler

Still at the same controller, you need to create a method for handling the authentication.

```ruby
def authenticate
  @user = User.find_signed(params[:token], purpose: :magic_link)

  if @user.present?
    sign_in(@user)

    flash[:success] = 'You have successfully logged in.'

    redirect_to root_path
  else
    flash[:info] = 'Link invalid or expired. Please try again.'

		# redirect to login page
    redirect_to new_user_session_path
  end
end
```

By using `find_signed` method, we can check whether the token is valid or not.

[https://api.rubyonrails.org/classes/ActiveRecord/SignedId/ClassMethods.html#method-i-find_signed](https://api.rubyonrails.org/classes/ActiveRecord/SignedId/ClassMethods.html#method-i-find_signed)

### Mailer to send the magic

Lastly, we need to create the Mailer class to send the magic link to the user. I will put the mailer method inside the `UserMailer`.

```ruby
class UserMailer < ApplicationMailer
  def magic_link(user_id, token)
    @user = User.find(user_id)
    @token = token

    mail(
      to: @user.email,
      subject: 'Magic link to sign in'
    )
  end
end
```

And here is the view

```ruby
<div>
    <h2>Hi!</h2>
</div>

<div>
  <p>Dear <%= @user.name %>,</p>
  <p>Click the link below to log in:</p>
  <%= link_to 'Login with link', authenticate_sign_in_links_url(token: @token) %>
  <p>This link will expire in 2 minutes.</p>
  <p>If you did not request this login link, please ignore this email.</p>
</div>
```

Congratulations! You've successfully integrated magic links into your application with only a few lines of code, eliminating the need for any external libraries.