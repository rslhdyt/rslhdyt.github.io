---
tags: ["rails", "sidekiq", "sidekiq-cron"]
date: 2024-03-01 00:00:00 +0700
description: Optimize Sidekiq Cron with a Single Worker Class for Rake Tasks. Simplify configuration, improve maintenance, and enhance testing capabilities. Streamline your Rails app's background job scheduling today!
published: true
collection: posts
category: blog
id: 1bd16ab6-e1ec-4588-b78a-4898d7df76e0
title: Streamline Your Sidekiq Cron Jobs with a Single Worker Class for Rake Task Invocation
created_time: 2024-03-02T06:37:00.000Z
cover: 
icon: 
last_edited_time: 2024-03-23T14:01:00.000Z
archived: false
created_by_object: user
created_by_id: 6b2c6a42-5dc5-4108-b726-4c02437b814d
last_edited_by_object: user
last_edited_by_id: 6b2c6a42-5dc5-4108-b726-4c02437b814d
---

![](https://media3.giphy.com/media/rytLWOErAX1F6/giphy.gif?cid=7941fdc631ow1c0odtfzkiifp1koqyox9jvx7qe7zat8fxhr&ep=v1_gifs_search&rid=giphy.gif&ct=g)

<em></em>

Running scheduled processes using Sidekiq Cron was a breeze. The setup process involves adding the Sidekiq Cron gem to the Gemfile and configuring the schedule.yml file to define cron schedule definitions. Additionally, the Sidekiq dashboard provides a convenient page to view the registered cron jobs.

Typically, when running a scheduled process, separate Sidekiq worker classes are created for each cron job, and within these workers, another service or rake task is invoked.

```yaml
daily_backup_worker:
  cron: "0 0 * * *"
  class: "DailyBackupWorker"
  description: "Run daily backup at midnight"
weekly_backup_worker:
  cron: "0 0 * * 0"
  class: "WeeklyBackupWorker"
  description: "Run weekly backup on Sunday at midnight"
```

For example, in the `DailyBackupWorker` class, the existing rake task command `db:backup` is called to trigger the database backup:

```ruby
require 'rake'
Rails.application.load_tasks

class DailyBackupWorker
  include Sidekiq::Worker

  def perform
    Rake::Task['db:backup'].execute
  end
end
```

Similarly, the `WeeklyBackupWorker` class would have similar code. However, it is possible to streamline the process by using a single class to handle rake task invocation within the Sidekiq cron worker.

Introducing the `InvokeRakeTaskWorker` class:

```ruby
require 'rake'
Rails.application.load_tasks

class InvokeRakeTaskWorker
  include Sidekiq::Worker

  def perform(command)
    Rake::Task[command['task']].execute(command['args'])
  end
end
```

And you can change your schedule.yml into like this.

```yaml
daily_backup_worker:
  cron: "0 0 * * *"
  class: "InvokeRakeTaskWorker"
  args:
    task: "db:backup"
  description: "Run daily backup at midnight"
weekly_backup_worker:
  cron: "0 0 * * 0"
  class: "InvokeRakeTaskWorker"
  args:
    task: "db:backup"
    args: "--weekly"
  description: "Run weekly backup on Sunday at midnight"
```

### Conclusion

That it’s, now you can use single class to invoke rake command inside sidekiq cron schedule.

using a single class to handle rake task invocation within the Sidekiq cron worker brings benefits such as code reusability, simplified configuration, flexibility, easier maintenance, and improved testing capabilities. Carefully evaluate the trade-offs and consider the specific requirements of your application before deciding on the approach to use.


