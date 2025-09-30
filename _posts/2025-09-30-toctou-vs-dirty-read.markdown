---
tags: ["rails", "race-condition", "journey", "ruby", "story"]
date: 2025-09-30 00:00:00 +0700
description: "Technical interviews often reveal gaps in our understanding. During a recent interview with a Japanese company, I confidently explained a race condition issue as a \"Dirty Read\" problem – a term I'd been misusing for years. The interviewer's follow-up questions led me to discover the real concept: TOCTOU (Time-of-Check to Time-of-Use). This interview not only corrected my misconception but reminded me why accepting interview opportunities is crucial for growth, even when you're not job hunting."
published: true
collection: posts
category: blog
id: 27e8e9d5-00d5-80b5-8e73-f5c8d54c07da
title: TOCTOU vs Dirty Read
created_time: 2025-09-30T12:14:00.000Z
cover: 
icon: 
last_edited_time: 2025-09-30T12:41:00.000Z
archived: false
created_by_object: user
created_by_id: 6b2c6a42-5dc5-4108-b726-4c02437b814d
last_edited_by_object: user
last_edited_by_id: 6b2c6a42-5dc5-4108-b726-4c02437b814d
---

## The Interview

Today is another reason and story to share about why I always encourage my fellow peers, subordinates, and friends to accept job offers and do as many job interviews as they can. Not only for the chance to get a new job, but in my opinion, the most important part are the experience the knowledge from the interview itself.

Today, I had a technical interview session with a company from Japan. Before the interview, I had to complete a take-home test, and the interviewer asked about my code and decisions based on the project.

The interviewer asked me to show my database design and asked some questions about the table design. To give you some perspective, let's say I have this table schema:

```ruby
# db/schema.rb
create_table "follows", force: :cascade do |t|
  t.bigint "follower_id", null: false
  t.bigint "followed_id", null: false
  t.datetime "created_at", null: false
  t.datetime "updated_at", null: false
  t.index ["followed_id"], name: "index_follows_on_followed_id"
  t.index ["follower_id", "followed_id"], name: "index_follows_on_follower_id_and_followed_id", unique: true
  t.index ["follower_id"], name: "index_follows_on_follower_id"
end
```

The interviewer asked: "Based on this database design, do you think it can handle when a user follows themselves?"

I said no, but I already handle that in the application layer with ActiveRecord validation. Here's the Follow model definition:

```ruby
# app/models/follow.rb
class Follow < ApplicationRecord
  belongs_to :follower, class_name: "User"
  belongs_to :followed, class_name: "User"
  has_many :sleeps, through: :followed

  validates :follower_id, presence: true
  validates :followed_id, presence: true
  validates :followed_id, uniqueness: { scope: :follower_id }
  validate :cannot_follow_self

  private

  def cannot_follow_self
    if follower_id == followed_id
      errors.add(:followed_id, "is the same as follower_id")
    end
  end
end
```

As you can see, there's a validation to prevent users from following themselves. And here's where the interesting part begins. The interviewer asked:

**Q: Do you think this ActiveRecord validation can handle race conditions?**

My answer was No! But since we have the database constraint, it will prevent duplicate records from being inserted into the database. The interviewer asked a follow-up question:

**Q: Why can't the ActiveRecord validation handle race conditions? Can you explain?**

I explained that it has a "Dirty Read" issue where two or more requests coming at the same time would cause ActiveRecord to perform validation by querying the follows table. Both requests would get the same result saying the record doesn't exist in the database yet, and the validation would pass.

The interview session ended and I was happy with my answers. Then came my learning moment.

## Today I Learned

After the interview, I tried to validate my answer by asking the same question to Claude and Google. Turns out my answer about the "Dirty Read" issue was not completely correct.

The correct term for why ActiveRecord cannot handle race conditions is **TOCTOU**. What is TOCTOU? TOCTOU stands for Time-of-Check to Time-of-Use.

Here's what happens. I'm using an email validation example to make it simple:

```ruby
# The race condition scenario:
# Time T1: Request A checks if "user@example.com" exists -> No, validation passes
# Time T2: Request B checks if "user@example.com" exists -> No, validation passes
# Time T3: Request A saves the record -> Success
# Time T4: Request B saves the record -> Success (DUPLICATE!)
```

Rails uniqueness validation works in two steps:

1. **SELECT** query to check if the value exists
2. **INSERT** query to save the record

Between these two steps, another process can insert the same value.

Here's reproducible code to simulate the race condition:

**The User model:**

```ruby
# app/models/user.rb
class User < ApplicationRecord
  validates :name, presence: true
  validates :email, uniqueness: true
  validates :name, length: { minimum: 3, maximum: 255 }
end
```

**The test script:**

```ruby
# race_condition_test.rb
require_relative 'config/environment'

email = "race_#{Time.now.to_i}@example.com"
pids = []

5.times do |i|
  # Using spawn instead of fork since fork doesn't work on macOS
  pids << spawn(
    "rails", "runner",
    "User.create!(email: '#{email}', name: 'Process #{i}')"
  )
end

pids.each { |pid| Process.wait(pid) }
puts "Total users with that email: #{User.where(email: email).count}"
```

**The result:**

```text
Total users with that email: 5
```

Five duplicate records despite our uniqueness validation!

## TOCTOU vs Dirty Read: Understanding the Difference

So what's the difference between TOCTOU and Dirty Read? Here's the explanation by Claude 😃

**Dirty Read** is a transaction isolation problem where one transaction reads uncommitted changes made by another transaction.

**TOCTOU** is a race condition where the state changes between checking a condition and acting on it.

### Side-by-Side Comparison

|Aspect|Dirty Read|TOCTOU|
|---|---|---|
|**What's the problem?**|Reading uncommitted data|State changes between check and use|
|**Transaction related?**|Yes, crosses transaction boundaries|No, can happen in single transaction|
|**Data validity**|Data may never exist (rollback case)|Data is real and committed|
|**When it occurs**|During concurrent transactions|Between sequential operations|
|**Prevention**|Transaction isolation level|Atomic operations, locks, or constraints|
|**Database can prevent it?**|Yes (Read Committed)|No (needs application logic)|

## The Lesson Learned

This interview taught me something valuable. I thought I understood race conditions, but I was mixing up different concepts. TOCTOU and Dirty Reads are both concurrency problems, but they're completely different beasts.

The key takeaways from this experience:

- **ActiveRecord validations are not enough** - they can't prevent race conditions
- **Database constraints are your best friend** - let the database do what it does best
- **Know your terminology** - understanding the difference between TOCTOU and Dirty Read helps you solve the right problem
- **Test with real concurrency** - don't just test sequential operations
- **Handle errors gracefully** - your users shouldn't see database constraint errors

Most importantly, this is why I love technical interviews. Even when you think you know something, there's always more to learn. Sometimes you need someone to ask the right question to realize you've been using the wrong term for years.

Next time someone asks you about race conditions, you'll know exactly what's happening under the hood. And more importantly, you'll know how to fix it properly.


