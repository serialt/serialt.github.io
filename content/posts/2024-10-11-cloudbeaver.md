+++

title = 'CloudBeaver'
date = 2024-10-11T20:45:27+08:00
draft = false

tags = ["redis"]
categories = ["DevOps"]

+++

## CloudBeaver 

web 版 DBeaver

docker-compose 安装

```yaml
version: "3"
services:
  cloudbeaver:
    image: dbeaver/cloudbeaver:24
    container_name: cloudbeaver
    restart: always
    ports:
      - "8978:8978"
    volumes: 
      - "/data/cloudbeaver:/opt/cloudbeaver/workspace"
    networks:
      - app
networks:
  app:   
    # 使用外部共享等网卡
    external: true 
```



terraform kubernetes

* https://github.com/serialt/terraform-module-cloudbeaver