---
tags: ["rails", "tutorial", "tips"]
date: 2023-10-03
description: Adding hooks to rake task
published: true
category: blog
id: 5d6110f6-1b6b-4939-a98d-2920c41cdd3e
title: Rails - Rake Task Hooks
created_time: 2022-08-08T04:18:00+00:00
cover: https://images.unsplash.com/photo-1621298516851-047f72b40bd7?ixlib=rb-1.2.1&q=80&cs=tinysrgb&fm=jpg&crop=entropy
icon: 
last_edited_time: 2023-11-17T07:32:00+00:00
archived: false
---

sample task 

```ruby
task :migrate_users do
  User.all.each { |u| u.migrate! }
end
```

to run the task `rake migrate_users`

what if we want to track elapsed time when run the task?

```ruby
# migrate_users.rake

task :migrate_users do
  start_at = Time.current

  User.all.each { |u| u.migrate! }

  end_at = Time.current

  puts "Elapsed time: #{end_at - start_at}"
end
```

Utilize rake hooks

ideal order should be like this

`before_hook` → `main_task` → `after_hook`

the actual order execution

`before_hook` → `after_hook` → `main_task`

how to fix

```ruby
# migrate_users.rake

task :before_hook
  @start_at = Time.current
end

task :after_hook
  at_exit do
    end_time = Time.current
    duration = end_time - @start_time
    puts "Elapsed time: #{duration}"
  end
end

task :migrate_users do
  User.all.each { |u| u.migrate! }
end

Rake::Task['numbers'].enhance [:before_hook, :after_hook]
```

applying as global hooks

```ruby
# Rakefile

task :before_hook do
  @start_time = Time.current
end

task :after_hook do
  at_exit do
    end_time = Time.current
    duration = end_time - @start_time
    puts "Elapsed time: #{duration}"
  end
end

tasks = Rake.application.tasks

tasks.each do |task|
  next if [Rake::Task['before_hook'],
           Rake::Task['after_hook']].include?(task)

  task.enhance([:before_hook, :after_hook])
end
```

---

reference: [https://t2013anurag.medium.com/rails-adding-global-hooks-for-rake-tasks-7faf0844364](https://t2013anurag.medium.com/rails-adding-global-hooks-for-rake-tasks-7faf0844364)

<br />
