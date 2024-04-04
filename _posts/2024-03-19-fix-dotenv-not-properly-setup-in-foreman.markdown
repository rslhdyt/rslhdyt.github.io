---
tags: ["rails", "foreman", "dotenv"]
date: 2024-03-19 00:00:00 +0700
description: Fix multiple .env file issue when running foreman
published: true
collection: posts
category: code_snippet
id: 71ec897f-95ea-4f3b-8745-33b6075a636d
title: Fix dotenv not properly setup in Foreman
created_time: 2024-03-20T02:44:00.000Z
cover: 
icon: 
last_edited_time: 2024-04-03T23:46:00.000Z
archived: false
created_by_object: user
created_by_id: 6b2c6a42-5dc5-4108-b726-4c02437b814d
last_edited_by_object: user
last_edited_by_id: 6b2c6a42-5dc5-4108-b726-4c02437b814d
---

When you're dealing with multiple environment files in your local environment, such as `.env.local` or `.env.staging`, and you're using Foreman to run the process, you might encounter an issue where the environment file doesn't load as expected.

To resolve this issue, you simply need to add the `-e` option with the `/dev/null` argument in your Foreman command.

```docker
exec foreman start -e /dev/null -f Procfile.dev "$@"
```

For a more detailed explanation of this issue, you can check out this conversation

[https://github.com/ddollar/foreman/pull/711](https://github.com/ddollar/foreman/pull/711)



