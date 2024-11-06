+++
title = "Разворачиваем AFFiNE аналог Notion"
draft = true
date = 2024-11-06
[taxonomies]
categories = ["blogs", "affine", "docker"]
tags = ["notion", "affine"]

+++

**AFFiNE** - это база знаний нового поколения, которая объединяет планирование, сортировку и создание контента. Позиционируется как альтернатива Miro, Notion и Monday. Чистый, интуитивно понятный дизайн. Интерфейс хорошо смотрится на десктопах и планшетах. В ближайших планах мобильное приложение и Канбан доска.

**Особенности**
* Поддержка MarkDown
  - AFFiNE поддерживает MarkDown для форматирования документов.
* Мультиязычность
  - AFFiNE предлагает поддержку нескольких языков, в том числе и русский
* Совместная работа
  - Из коробки доступно облако для личного использования на разных девайсах, или для работы в команде
* Фокус на приватности
  - AFFiNE использует в первую очередь локальную базу, облако можно не подключать. Чтобы поделиться контентом, его можно экспортировать в Html или MarkDown

![affine_start.png](affine_start.png)

## Установка AFFiNE в docker

```yaml
services:
  affine:
    image: ghcr.io/toeverything/affine-graphql:stable
    container_name: affine_selfhosted
    command:
      ['sh', '-c', 'node ./scripts/self-host-predeploy && node ./dist/index.js']
    ports:
    # CHANGE YOUR PORTS HERE IF NEEDED PORT:PORT_INSIDE_CONTAINER
      - '3010:3010'
      - '5555:5555'
    depends_on:
      redis:
        condition: service_healthy
      postgres:
        condition: service_healthy
    volumes:
      # CHANGE YOUR PATH HERE FOR the custom configuration
      - /mnt/user/appdata/affine-app:/root/.affine/config
      # CHANGE YOUR PATH HERE FOR blob storage
      - /mnt/user/appdata/affine-app:/root/.affine/storage
    logging:
      driver: 'json-file'
      options:
        max-size: '1000m'
    restart: unless-stopped
    environment:
      - NODE_OPTIONS="--import=./scripts/register.js"
      - AFFINE_CONFIG_PATH=/root/.affine/config
      - REDIS_SERVER_HOST=redis
      # REPLACE MY_DB_PWD_TOP_SECRET BY YOUR PWD
	- DATABASE_URL=postgres://affine:MY_DB_PWD_TOP_SECRET@postgres:5432/affine
      - NODE_ENV=production
      # REPLACE YOUR_EMAIL BY YOUR EMAIL
      - AFFINE_ADMIN_EMAIL=YOUR_EMAIL
      # REPLACE YOUR_PWD BY YOUR PWD
      - AFFINE_ADMIN_PASSWORD=YOUR_EMAIL
  redis:
    image: redis
    container_name: affine_redis
    restart: unless-stopped
    volumes:
    # CHANGE YOUR PATH HERE
      - /mnt/user/appdata/affine-redis:/data
    healthcheck:
      test: ['CMD', 'redis-cli', '--raw', 'incr', 'ping']
      interval: 10s
      timeout: 5s
      retries: 5
  postgres:
    image: postgres
    container_name: affine_postgres
    restart: unless-stopped
    volumes:
    # CHANGE YOUR PATH HERE
      - /mnt/user/appdata/affine-db:/var/lib/postgresql/data
    healthcheck:
      test: ['CMD-SHELL', 'pg_isready -U affine']
      interval: 10s
      timeout: 5s
      retries: 5
    environment:
      POSTGRES_USER: affine
      # REPLACE MY_DB_PWD_TOP_SECRET BY YOUR PWD
      POSTGRES_PASSWORD: MY_DB_PWD_TOP_SECRET
      POSTGRES_DB: affine
      PGDATA: /var/lib/postgresql/data/pgdata
```

Поменяйте параметры: **MY_DB_PWD_TOP_SECRET**, **YOUR_EMAIL** и **YOUR_EMAIL** на свои.

Запустите командой: ```docker-compose up -d```

## Добавление настроек в nginx

```sh
server {
    listen 80;
    server_name affine.malexple.ru;
    access_log /var/log/nginx/nginx.local-access.log;
    error_log /var/log/nginx/nginx.local-error.log;
    fastcgi_param HTTPS on;

    location / {
        proxy_pass http://192.168.30.32:3010;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

## Синхронизация с облаком
При первом запуске вам будет предложено синхронизироваться с облаком. Смело нажимайте синхронизироваться т.к. это ваш личный сервер на котором будут храниться данные.