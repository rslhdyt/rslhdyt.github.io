---
tags: ["tips", "feature-toggle"]
date: 2023-11-21 00:00:00 +0700
description: In this guide, we'll dig into this scenario, explaining how Flipper acts as the bridge between your frontend and backend. By the end, you'll know how to use Flipper to take charge of your feature toggles, making your application more adaptable and user-friendly. Let's make managing feature toggles easy with Flipper!
published: true
collection: posts
category: blog
id: 5b52466d-13e7-45dd-9126-348383b3be64
title: "Flipper: Retrieve a List of Features for an Actor"
created_time: 2023-11-20T23:58:00.000Z
cover: 
icon: 
last_edited_time: 2024-03-23T14:01:00.000Z
archived: false
created_by_object: user
created_by_id: 6b2c6a42-5dc5-4108-b726-4c02437b814d
last_edited_by_object: user
last_edited_by_id: 6b2c6a42-5dc5-4108-b726-4c02437b814d
---

![](https://images.unsplash.com/photo-1591106167857-f1f9257e671b?ixlib=rb-4.0.3&q=85&fm=jpg&crop=entropy&cs=srgb)

<em>Image from [Ranae Smith](https://unsplash.com/photos/flock-of-birds-flying-over-the-sea-during-daytime-UDlXygG0pgA?utm_source=63921&utm_medium=referral) from Unsplash</em>

When your frontend is a SPA, dealing with feature toggles can be confusing. The challenge is making sure the frontend can access and control these toggles without making things too complicated.

Let's say you want to use Flipper to centralize your feature toggles. You could use the backend or Flipper UI as a sort of control center. Your aim? To effortlessly display all available feature toggles for a specific user. You might decide to showcase these toggles on a user's profile, reachable through an endpoint like /profile.

In this guide, we'll dig into this scenario, explaining how Flipper acts as the bridge between your frontend and backend. By the end, you'll know how to use Flipper to take charge of your feature toggles, making your application more adaptable and user-friendly. Let's make managing feature toggles easy with Flipper!

The profile endpoint

```ruby
def profile
	render json: current_user
end
```

```ruby
{
  "name": "Foo",
  "email": "foo@example.com",
  ...
}
```

### Retrieve a list of features by actor

```ruby
Flipper.features.inject({}) do |hash, feature|
  hash[feature.key] = featuer.enabled? ACTOR
  hash
end
```

In this code, ensure that you replace `**ACTOR**` with the appropriate user or actor identifier in your application. Additionally, consider adding comments for clarity in the code.

<br />

```ruby
def profile
	render json: current_user_with_features
end

private

def current_user_with_features
	current_user.merge({ features: user_features })
end

def user_features
	Flipper.features.inject({}) do |hash, feature|
	  hash[feature.key] = feature.enabled? current_user
    hash
  end
end
```

### Output

```ruby
{
  "name": "Foo",
  "email": "foo@example.com",
  "features": {
    "ft_document_upload": true,
    "ft_bulk_export": false,
    ...
  }
}
```


