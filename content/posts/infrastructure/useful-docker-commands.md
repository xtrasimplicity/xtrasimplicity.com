---
title: "Useful Docker Commands"
date: 2018-08-02T13:50:17+10:00
draft: false
tags: ['docker', 'tips']
categories: ['Docker']
---
The following are some useful docker commands that I haven't yet committed to memory.

## Delete all containers
```bash
$(echo docker ps -aq) | while read -r line; do docker rm "${line}"; done
```

## Delete ALL docker-related data (images, containers, etc)
**WARNING**: As with `rm -rf` on Linux, this action is __not__ reversible. Take care when using this command.

```bash
docker system prune -a -f
```
