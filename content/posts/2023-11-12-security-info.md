+++
title = 'security-info'
date = 2023-11-12T10:29:27+08:00
draft = false

tags = ["security","cve"]
categories = ["安全信息"]

+++

## 漏洞信息

云服务商安全服务公告

| 云服务商         | 链接                                                                                    |
| ---------------- | --------------------------------------------------------------------------------------- |
| cve官网          | https://cve.mitre.org/                                                                  |
| 阿里云           | https://help.aliyun.com/noticelist/9213612.html?spm=a2c4g.789004748.n2.3.cddb4c07NBt9Rl |
| 华为云           | https://www.huaweicloud.com/notice.securecenter.html                                    |
| 腾讯云           | https://cloud.tencent.com/announce?categorys=21&page=1                                  |
| 阿里云漏洞数据库 | https://avd.aliyun.com/                                                                 |
| 国家信息漏洞库   | http://www.cnnvd.org.cn/                                                                |



常用软件官方安全公告链接

| 名称  | 链接                                         |
| ----- | -------------------------------------------- |
| redis | https://github.com/redis/redis/security      |
| nginx | http://nginx.org/en/security_advisories.html |
| httpd | https://httpd.apache.org/security            |



## 安全服务推送

github: https://github.com/zema1/watchvuln

```yaml
version: "3"
services:
  watchvuln:
    image: zemal/watchvuln
    container_name: watchvuln
    hostname: watchvuln
    restart: always
    environment:
      DINGDING_ACCESS_TOKEN: cd316d9dxxxxxxxxxxxxxxxxxxxxxxx
      DINGDING_SECRET: SECa87a39xxxxxxxxxxxxxxxxxxxxxxxxxxxx
      LARK_ACCESS_TOKEN: 1ddfb805-xxxxxxxxxxxxxxxxxxxxxxxxxxxx
      LARK_SECRET: GUUKIrxxxxxxxxxxxxxxxxxxxxxxxxxxxx
      INTERVAL: 30m 
    volumes:
      - "/etc/localtime:/etc/localtime:ro"
```

