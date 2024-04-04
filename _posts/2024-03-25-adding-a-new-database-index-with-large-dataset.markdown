---
tags: ["database-locking", "database-migration", "rails", "TIL"]
date: 2024-03-25 00:00:00 +0700
published: false
collection: posts
category: blog
id: 654248a4-05a4-40a5-89af-04f8b71c2ee2
title: Adding a new database index with large dataset
created_time: 2024-03-24T23:58:00.000Z
cover: 
icon: 
last_edited_time: 2024-04-03T15:46:00.000Z
archived: false
created_by_object: user
created_by_id: 6b2c6a42-5dc5-4108-b726-4c02437b814d
last_edited_by_object: user
last_edited_by_id: 6b2c6a42-5dc5-4108-b726-4c02437b814d
---

Have you handle a project with a huge data? like ten of millions or even hundred millions. If so, how you manage database migration, such as add new field into an existing table? Do you want hear my experience? let me start my story.

---

What will happened when you create a database migration, like add a new field to the existing table that have ten of million or even hundred million records? There are two scenario depends what are strategies do you used.


