---
layout: default
title: Docker Cheats
permalink: /docker-cheats
---

# Docker Cheats
_2026-04-26_

## Build
```bash
docker build -t app .
```

## Run
```bash
docker run -p 8000:8000 app
```

## Run (detached)
```bash
docker run -d -p 8000:8000 app
```

## List containers
```bash
docker ps
docker ps -a
```

## Stop container
```bash
docker stop <container_id>
```

## Remove container
```bash
docker rm <container_id>
```

## Remove image
```bash
docker rmi <image_id>
```

## Exec into container
```bash
docker exec -it <container_id> bash
```

## Logs
```bash
docker logs -f <container_id>
```

## Compose
```bash
docker compose up
docker compose up -d
docker compose down
```

## Clean up
```bash
docker system prune
```
