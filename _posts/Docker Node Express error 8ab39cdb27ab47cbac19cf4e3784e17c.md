---
layout: post
title:  "Docker Node Express error"
tags: Docker TroubleShooting
---

## 개요..

js pollution 테스트 환경 구축하려고 docker로 node express를 올리는데 자꾸 volume만 마운트하면 `Cannot find module ‘express’` 란 에러가 발생합니다.

```docker
docker run <image tag>

> start
> node index.js

node:internal/modules/cjs/loader:927
  throw err;
  ^

Error: Cannot find module 'express'
Require stack:
- /index.js
```

## 해결

이유는 mount 하면서 npm install 된 node_moudles 폴더가 사라지기 때문에 모듈이 삭제 됩니다.

그래서 mount 이후에 따로 node_modules를 따로 마운트해주면 해결할 수 있습니다.

```yaml
version: '2.3'
services:
  express:
    build: ./
    container_name: express
    volumes:
      - .:/app
      - /app/node_modules
    ports:
      - "1111:8888"
```

## R**eference**

[https://www.reddit.com/r/docker/comments/muqfds/newbie_in_docker_getting_error_cannot_find_module/](https://www.reddit.com/r/docker/comments/muqfds/newbie_in_docker_getting_error_cannot_find_module/)