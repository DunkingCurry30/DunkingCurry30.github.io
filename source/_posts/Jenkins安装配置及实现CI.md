---
title: Markdown 常见问题解决
date: 2022/6/2 20:00:00
tags: 
  - jenkins
  - docker
categories: 
  - DevOps
---

# Jenkins安装配置



## 1. docker-compose 启动 Jenkins

创建 `jenkins` 目录，在目录下编辑 `docker-compose.yml` 

```bash
mkdir -p /opt/docker/jenkins && cd /opt/docker/jenkins
#创建data目录并赋权，不然会启动失败
mkdir data && chmod 777 data
#创建docker-compose配置
vim docker-compose.yml
```

内容如下

```yaml
version: '3'
services:
    jenkins:
        image: jenkins/jenkins  #jenkins镜像
        container_name: jenkins  #容器名
        restart: always
        ports:
        	- '8080:8080'  #发布端口
            - '50000:50000'  #基于 JNLP 的 Jenkins 代理通过 TCP 端口 50000 与 Jenkins master 进行通信
        volumes:
        	- ./data:/var/jenkins_home  #工作目录挂载到外部服务器
```

执行命令启动 `jenkins` 容器

```bash
docker-compose up -d jenkins
```



## 2. Jenkins 安装

通过 `ip:端口` 访问web端进行安装

![1657024634835](../blog-assets/Jenkins安装配置及实现CI/1657024634835.png)

查看日志获取初始密码

```bash
cat /opt/docker/jenkins/data/secrets/initialAdminPassword 
```

选择推荐插件，等待安装完成

![1657025555952](../blog-assets/Jenkins安装配置及实现CI/1657025555952.png)

创建一个新的管理员账户

![1657025661533](../blog-assets/Jenkins安装配置及实现CI/1657025661533.png)

安装成功

![1657025778183](../blog-assets/Jenkins安装配置及实现CI/1657025778183.png)