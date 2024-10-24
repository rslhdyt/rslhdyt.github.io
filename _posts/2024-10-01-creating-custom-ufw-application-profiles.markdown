---
tags: ["tutorial", "security", "vps"]
date: 2024-10-01 00:00:00 +0700
description: "Learn how to create custom UFW (Uncomplicated Firewall) application profiles on Linux servers. This quick guide shows you how to secure your VPS with specific port rules - perfect for developers managing custom applications. Simple steps, real examples, and best practices included. #Linux #Security #DevOps"
published: true
collection: posts
category: blog
id: 1288e9d5-00d5-8000-b90b-d2171a410107
title: Creating Custom UFW Application Profiles
created_time: 2024-10-23T02:55:00.000Z
cover: 
icon: 
last_edited_time: 2024-10-24T07:30:00.000Z
archived: false
created_by_object: user
created_by_id: 6b2c6a42-5dc5-4108-b726-4c02437b814d
last_edited_by_object: user
last_edited_by_id: 6b2c6a42-5dc5-4108-b726-4c02437b814d
---

When setting up a VPS for your pet projects, you often need to allow specific applications through UFW (Uncomplicated Firewall). While UFW comes with some pre-defined application profiles, you might need custom ones for your specific use case.

Custom UFW application profiles can be created by adding new files in `/etc/ufw/applications.d/`

### Step-by-Step Guide

1. **Create a new profile file**
```shell
sudo nano /etc/ufw/applications.d/myapp
```

2. **Add your application definition**
```text
[MyApp]
title=My Custom Application
description=Custom ports for my application
ports=8080,8081/tcp|9090/udp
```

3. **Update UFW**
```shell
sudo ufw app update MyApp
```

4. **Verify the profile**
```shell
sudo ufw app info MyApp
```

5. **Enable the application**
```shell
sudo ufw allow MyApp
```

Remember to reload UFW after adding profiles:

```shell
sudo ufw reload
```

### Real Example

Here's an actual profile I use for a Node.js application with WebSocket support:

```text
[NodeApp]
title=Node.js Full Stack App
description=Node.js application with HTTP and WebSocket ports
ports=3000/tcp|3001/tcp|8080/tcp
```

### Understanding the Profile Format

- `[MyApp]`: Profile name (used in UFW commands)
- `title`: Short description
- `description`: Longer explanation
- `ports`: Port definitions
	- Use comma (,) to separate multiple ports
	- Use pipe (|) to separate TCP and UDP
	- Add /tcp or /udp to specify protocol

<br />


