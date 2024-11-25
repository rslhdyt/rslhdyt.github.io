---
tags: ["webhookdump", "rails"]
date: 2024-03-15 00:00:00 +0700
description: After setting up the database structure, it's time to build the core models that power WebhookDump.
published: true
collection: posts
category: blog
id: 1498e9d5-00d5-80d6-8cf9-d186edb3b1be
title: "Building the Core: Webhook and Request Models"
created_time: 2024-11-25T06:54:00.000Z
cover: 
icon: 
last_edited_time: 2024-11-25T08:44:00.000Z
archived: false
created_by_object: user
created_by_id: 6b2c6a42-5dc5-4108-b726-4c02437b814d
last_edited_by_object: user
last_edited_by_id: 6b2c6a42-5dc5-4108-b726-4c02437b814d
---

💡 See the code for this post on the 🔗 core-models branch.

After creating our database structure in the previous chapter, it's time to add logic to our models. The models are the core of our application, handling data validation and relationships.

## The Webhook Model

Let's start with the `Webhook` model. This model needs to:

- Generate a unique UUID for new webhooks
- Check if a webhook has expired
- Manage its relationship with webhook requests

Here's how I implemented these requirements:

```ruby
class Webhook < ApplicationRecord
  has_many :webhook_requests, dependent: :destroy

  validates :uuid, presence: true, uniqueness: true
  validates :expired_at, presence: true

  before_validation :set_uuid, on: :create

  def expired?
    Time.current.to_date > expired_at
  end

  private

  def set_uuid
    self.uuid = SecureRandom.uuid if uuid.blank?
  end
end

```

A few interesting things about this model:

1. The `has_many` relationship tells Rails that one webhook can have multiple webhook requests
2. `dependent: :destroy` ensures all related webhook requests are deleted when we delete a webhook
3. We use a `before_validation` callback to automatically set the UUID when creating a new webhook
4. The `expired?` method helps us check if a webhook can still receive requests

## The WebhookRequest Model

Next, I built the `WebhookRequest` model to store incoming HTTP requests. This model needs to:

- Store all the details of an incoming request
- Belong to a webhook

Here's the implementation:

```ruby
class WebhookRequest < ApplicationRecord
  belongs_to :webhook

  # Convert headers and query params to JSON before saving
  before_save :ensure_json_format

  private

  def ensure_json_format
    self.headers = headers.to_json unless headers.is_a?(String)
    self.query_params = query_params.to_json unless query_params.is_a?(String)
  end
end

```

The interesting parts of this model:

1. `belongs_to :webhook` sets up the relationship with the parent webhook
2. We validate all fields because each piece of request information is important
3. The `ensure_json_format` callback makes sure our headers and query parameters are stored as JSON strings

## Testing the Models

Let's test our models in the Rails console:

```ruby
# Create a new webhook that expires in 7 days
webhook = Webhook.create!(expired_at: 7.days.from_now)

# Create a webhook request
request = webhook.webhook_requests.create!(
  ip: "127.0.0.1",
  url: "https://webhookdump.link/abc123",
  method: "POST",
  host: "webhookdump.link",
  headers: {"Content-Type" => "application/json"},
  query_params: {"key" => "value"},
  payload: '{"message": "Hello World"}'
)

# Check if it worked
puts "Webhook UUID: #{webhook.uuid}"
puts "Request count: #{webhook.webhook_requests.count}"
puts "Is expired? #{webhook.expired?}"

```

With these models in place, our application can now:

1. Create webhooks with unique identifiers
2. Store incoming webhook requests with all their details
3. Check if webhooks are still valid
4. Maintain proper relationships between webhooks and their requests

In the next chapter, we'll build the controllers to handle incoming webhook requests and display them to users.

Stay tuned! 🚀

