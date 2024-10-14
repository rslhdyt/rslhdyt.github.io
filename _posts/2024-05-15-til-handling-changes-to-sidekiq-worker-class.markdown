---
tags: ["rails", "sidekiq", "sidekiq-cron", "TIL"]
date: 2024-05-15 00:00:00 +0700
description: Today I learned a valuable lesson about the intricacies of modifying Sidekiq worker classes in our Ruby on Rails application. What started as a routine code deployment turned into an unexpected adventure, revealing the hidden complexities of Sidekiq's job persistence mechanism.
published: true
collection: posts
category: blog
id: 291d80b9-bf4c-474d-a971-2e318039ee31
title: "TIL: Handling Changes to Sidekiq Worker Class"
created_time: 2024-05-15T14:15:00.000Z
cover: 
icon: 
last_edited_time: 2024-10-14T16:09:00.000Z
archived: false
created_by_object: user
created_by_id: 6b2c6a42-5dc5-4108-b726-4c02437b814d
last_edited_by_object: user
last_edited_by_id: 6b2c6a42-5dc5-4108-b726-4c02437b814d
---

This articles documents two related issues encountered by Sidekiq workers during my recent code deployments to the production environment. Both issues highlight the importance of understanding Sidekiq's behavior when modifying worker classes or their arguments.

The first issue involved the removal of a scheduled Sidekiq worker class, while the second issue pertained to modifying the arguments accepted by a worker class. In both cases, errors occurred due to Sidekiq's handling of existing jobs in the queue.

The report aims to analyze the root causes of these issues, outline the remediation steps taken, and propose preventive measures to avoid similar occurrences in the future. Additionally, it summarizes the key lessons learned to improve the team's development practices and ensure a more robust Sidekiq implementation.

## Problem Statement

As mentioned in the introduction, this report covers two different cases but with the same behavior.

1. The deleted sidekiq cron scheduler still exists in the cron scheduler list
We have a sidekiq worker class called ResetDocQuotaWorker. That worker class is responsible for calculating and resetting user document quota. The worker running daily at 00:00 UTC timezone time. There were reports that some user document quotas were not running as expected, and after checking with the engineer, we found the issue that the data processed by the worker was too big and needed refactoring.

The engineer split the worker into several classes to make the data smaller and chunked into separate groups of subscription types. So the new classes are ResetFreeDocQuotaWorker and ResetPaidDocQuotaWorker. Therefore the old class ResetDocQuotaWorker was removed from the schedule.yml

After the changes were deployed to production, the new worker appeared in the sidekiq-cron list scheduler. However, the old class ResetDocQuotaWorker still exists and running as usual, causing an error in our monitoring report.

2. The worker class stopped working after the class argument changed
The next involved the SignGlobalWorker class. The error was raised due to the sidekiq worker job failing to fetch the parameters from the class. Upon checking the root cause there is a change in the arguments of SignGlobalWorker. Previously, the class accepted 4 arguments, user_id, envelope_id, signature, initial. And there is a need to add 1 more argument signature_id to support business needs. However, the rubocop said that the 5 arguments on a single method violated the rule. Thus, the engineer decided to refactor the argument into a hash containing the keys and the values of the param.


### Root Cause Analysis

Both issues in the previous problem statement section have the same behavior. Which was the sidekiq job client still processing the old behavior before the changes. The root of the problem is that Sidekiq persists jobs in Redis, and it doesn't automatically update or remove these persisted jobs when you make changes to your worker classes or schedule file. Here are the detailed explanations of both problems.

1. **Removing a worker class from the schedule file**: When you remove a worker class from the schedule.yml file, Sidekiq doesn't automatically remove the scheduled instances of that job. This is because Sidekiq persists in scheduled jobs in Redis, and it doesn't check for removed jobs in the schedule file. It only schedules new jobs that it finds in the file. So, even if you remove a job from the schedule.yml file, the previously scheduled instances of that job will still exist in Redis and will be executed at their scheduled times.
2. **Changing the arguments of a worker class**: When you enqueue a job in Sidekiq, it serializes the job's class name and arguments and stores them in Redis. When it's time to execute the job, Sidekiq fetches the job from Redis, deserializes it, and calls the perform method on the job's class with the deserialized arguments. If you change the arguments that the perform method expects (for example, from four arguments to a single hash), but there are still jobs in Redis that were enqueued with the old arguments, Sidekiq will try to call the perform method with the old arguments, causing an ArgumentError.

### Remediation Steps

This section tries to explain how we should handle the changes in the sidekiq worker.

### Handling deletion of existing sidekiq-cron schedule

If you need to remove the existing sidekiq cron schedule class follow these steps:

1. Disable instead remove the config
Sidekiq cron job properties support optional properties for toggling to enable or disable scheduled jobs. The key name is status with the value disabled or enabled, the default value is enabled.

So instead of removing the job from the schedule.yml we can disable the job by setting the job status to disabled. This setup prevents the scheduled job from running after the changes are deployed to the server.

```text
{
  # MANDATORY
  'name' => 'name_of_job', # must be uniq!
  'cron' => '1 * * * *',  # execute at 1 minute of every hour, ex: 12:01, 13:01, 14:01
  ...
  'class' => 'MyClass',
  # OPTIONAL
  ...
  'status' => 'disabled'
}
```

2. Remove the job via the Rails console
After the changes were deployed to production and no major issue was raised. We can delete the old job by executing the below command.

```ruby
job = Sidekiq::Cron::Job.find('name_of_the_job')
# example: Sidekiq::Cron::Job.find('reset_doc_quota_worker')

# Remove the job
job.destroy if job
```


### Handling sidekiq worker argument changes

This one is a bit tricky, but the rule of thumb is that the sidekiq worker changes should be backward compatible. It means that the latest changes to the existing worker should not affect the previous version of the worker and vice versa.

```ruby
module BulkSigns
  class SignGlobalWorker
    include Sidekiq::Worker

    def perform(user_id, envelope_id, initial, signature)
      # do something
    end
  end
end
```

As you can see in the previous code snippet, the SignGlobalWorker class accepts 4 arguments. So if you need to update or add more arguments follow these rules.

1. **Do not change** the order of the existing arguments
```text
❌
def perform(user_id, signature, envelope_id, initial)
  # do something
end
```

2. **Do not remove** the existing arguments
```text
❌
def perform(user_id, envelope_id, initial)
  # do something
end
```

3. **Add default value** for a newly added argument
```text
✅
def perform(user_id, envelope_id, initial, signature, signature_id = nil)
  # do something
end
```


If the refactoring is inevitable, you can consider creating a new worker class instead of modifying the existing one. This option will prevent the ArgumentError from being raised because the old worker will process existing parameters from Redis and eventually stop running. After all, no new job is dispatched to the old job.

```text
module Fruit
  module V2
    class BananaWorker
      include Sidekiq::Worker

      def perform(params)
        # do something
      end
    end
  end
end

Fruit::V2::BananaWorker.perform_async({
  user_id: USER_ID,
  envelope_id: ENVELOPE_ID,
  initial: INITIAL,
  signature: SIGNATURE,
  signature_id: SIGNATURE_ID
})
```

## Summary

The issues documented in this report highlight the importance of understanding Sidekiq's behavior and taking necessary precautions when modifying worker classes or their arguments. **Failure to do so can lead to errors, job failures, and potential data inconsistencies or loss.**

The root cause analysis revealed that Sidekiq persists jobs in Redis and does not automatically update or remove these persisted jobs when changes are made to worker classes or the schedule file. This behavior can cause issues when removing scheduled worker classes or modifying the arguments of existing worker classes.

Adhering to these principles will maintain a robust and reliable Sidekiq implementation for efficient background job processing.


