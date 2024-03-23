---
tags: ["rails", "tutorial"]
date: 2023-11-24 00:00:00 +0700
published: true
collection: posts
category: blog
id: adf9890d-36ff-4374-98d0-38aa9d9d5ddd
title: "Rails Authentication: Setup Omniauth using Devise"
created_time: 2023-11-13T02:31:00.000Z
cover: 
icon: 
last_edited_time: 2024-03-23T14:01:00.000Z
archived: false
created_by_object: user
created_by_id: 6b2c6a42-5dc5-4108-b726-4c02437b814d
last_edited_by_object: user
last_edited_by_id: 6b2c6a42-5dc5-4108-b726-4c02437b814d
---

![](https://images.unsplash.com/flagged/photo-1561023367-50a6e054d890?ixlib=rb-4.0.3&q=85&fm=jpg&crop=entropy&cs=srgb)

<em></em>

Introduction

Social login or omniauth is one of must have authentication in web application. It helps our to onboard the users who is reluctant to fill the registration form and register or login by using the account from another services like google or github.

What is omniauth

Omniauth is one of authentication method that allowed a user logged in or register to our application with the credentials or user information from another service. Omniauth use Oauth protocol to process the authentication and get user information.

Implementing Omniauth in rails application using devise

Devise has by default has a setting to enable or disable the omniauth authentication. However, we need to install separated oauth providers that we want to use. In this tutorial, I will use google as the oauth provider.

Obtaining google credentials for omniauth

I assumed you have google cloud platform account and project. those information required to create the oauth credentials.

Go to google console and go to APIs & Service menu and select submenu Credentials

![](/assets/images/posts/4667ee20-697a-4111-8e32-dc2aba35ed99-Untitled.png)

<em></em>

On the credentials page, click the Create Credentials button and choose the Oauth Client ID dropdown option

![](/assets/images/posts/871ea8c2-828c-46a4-90aa-6a2fff175a3a-Untitled.png)

<em></em>

Select `Web applications` and provide a name. Under the `Authorized redirect URIs` add the path to callback endpoint. I put `http://localhost:3000/auth/google_oauth2/callback`.

![](/assets/images/posts/1b58e32c-53c4-4b61-aafa-8e78d7a64532-Untitled.png)

<em></em>

After the form submited, the modal will be shown, copy Client ID and Client Secret and put it in environment config file like this.

```ruby
GOOGLE_CLIENT_ID=YOUR_CLIENT_ID
GOOGLE_CLIENT_SECRET=YOUR_CLIENT_SECRET
```

![](/assets/images/posts/0168a681-8603-402f-838a-11156a4f8926-Untitled.png)

<em></em>

Installing google provider gem

To enable the login using google we need to install additional gem, add this gem to your Gemfile

```ruby
gem 'omniauth-google-oauth2'
```

Don’t forget to run bundle install after adding new gem in Gemfile.

Update devise config

```ruby
Devise.setup do |config|
  ...

  # uncomment this line below
  # config.omniauth :google_oauth2, CLIENT_ID, CLIENT_SECRET, {}
  config.omniauth :google_oauth2, ENV.fetch('GOOGLE_CLIENT_ID', nil), ENV.fetch('GOOGLE_CLIENT_SECRET', nil), {}

  ...
end
```


