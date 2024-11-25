---
tags: ["docker", "rails", "kamal"]
date: 2024-10-25 00:00:00 +0700
description: Learn how to set CPU and memory limits in Kamal 2 for multi-app deployments. Step-by-step guide for configuring resource constraints in application, accessory, and proxy containers.
published: true
collection: posts
category: blog
id: 12a8e9d5-00d5-8063-8a11-eb7109beb0d7
title: Setting Resource Limits in Kamal 2
created_time: 2024-10-25T15:13:00.000Z
cover: 
icon: 
last_edited_time: 2024-10-25T15:44:00.000Z
archived: false
created_by_object: user
created_by_id: 6b2c6a42-5dc5-4108-b726-4c02437b814d
last_edited_by_object: user
last_edited_by_id: 6b2c6a42-5dc5-4108-b726-4c02437b814d
---

One of the most requested features in Kamal 2 is the ability to easily deploy multiple applications on a single server. While Kamal makes this process straightforward, running multiple apps on shared hardware comes with an important consideration: resource management. You need to distribute CPU and memory wisely to ensure each application gets its fair share without starving others.

## Understanding Kamal's Container Architecture

Kamal uses Docker under the hood, specifically the `docker run` command to manage containers. This means we can leverage Docker's resource limitation options, but the configuration method differs between:

- Servers or Roles
- Accessory (like Redis, PostgreSQL)
- Proxy (Kamal Proxy)

## Setting Resource Limits for Roles and Accessories

For your main application and accessories, add options key to your `deploy.yml`:

```yaml
# config/deploy.yml

servers:
	host: HOST
  options:
    cpus: "1"     # Use 1 CPU core
    memory: 512M  # Limit memory to 512MB

accessories:
  redis:
    options:
      cpus: "0.5"   # Half a CPU core
      memory: 256M  # 256MB memory limit
```

## Setting Resource Limits for Proxy

The proxy container (Kamal proxy) requires a different approach. We need to use the Kamal CLI to set Docker options:

```shell
# Set CPU and memory limits for proxy
kamal proxy boot_config set --docker-options="cpus=0.5 memory=256M"

# Verify current proxy settings
kamal proxy boot_config get

# Restart the proxy to apply the changes
kamal proxy reboot
```

## Pro Tips 💡

1. **Monitor Before Setting**: Deploy without limits first and monitor actual usage to set appropriate values.
2. **Safety Buffer**: Always add ~20% buffer to your observed memory usage and leave some headroom for the host system.

---

Need to learn more? Check out:

- [Kamal Documentation](https://kamal-deploy.org/)
- [Docker Resource Constraints](https://docs.docker.com/config/containers/resource_constraints/)


