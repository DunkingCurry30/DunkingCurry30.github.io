---
title: JFrog 的搭建与使用
date: 2022/8/7 20:00:00
tags: 
  - tools
categories: 
  - DevOps
  - 环境搭建
---

> JFrog 官方安装文档：  [Installing Artifactory - JFrog Documentation](https://www.jfrog.com/confluence/display/JFROG/Installing+Artifactory#InstallingArtifactory-DockerComposeInstallation) 

# 一、JFrog Artifactory

## 1. docker-compose 启动 JFrog Artifactory

```bash
mkdir -p /opt/docker/jfrog/artifactory && cd /opt/docker/jfrog/artifactory
#创建data目录并赋权，不然会启动失败
mkdir data && chmod 777 data
#创建docker-compose配置
vim docker-compose.yml
```

 内容如下 ：

```yaml
version: '3'
services:
    jfrog-oss:
        image: docker.jfrog.io/jfrog/artifactory-oss #社区版，下载较慢
        container_name: jfrog-oss  #容器名
        restart: always
        ports:
        	- '8661:8081'  # Rest Api端口，用作上传拉取制品包
        	- '8662:8082'  # web访问端口，必须开放，否自报错404
        volumes:
        	- ./data:/var/opt/jfrog/artifactory  #工作目录挂载到外部服务器
```

执行命令启动 `jfrog-oss` 容器

```bash
docker-compose up -d jfrog-oss
```



## 2. Artifactory 的使用

通过 ` http://SERVER_ HOSTNAME:8662/ui/` 访问web端，输入默认用户名密码 `admin/password` ：

![1659863206702](../blog-assets/JfrogArtifactory搭建/1659863206702.png)

登录成功后修改管理员密码，即可正常使用。



# 二、JFrog Container Registry

因为目前 `Artifactory` 社区版仅支持`Maven`等五类仓库，暂时不支持 `Docker` ，因此需要安装 JCR（JFrog Container Registry）作为 `Docker` 镜像仓库。

## 1. docker-compose 启动 JCR

```bash
mkdir -p /opt/docker/jfrog/jcr && cd /opt/docker/jfrog/jcr
#创建data目录并赋权，不然会启动失败
mkdir data && chmod 777 data
#创建docker-compose配置
vim docker-compose.yml
```

内容如下 ：

```yaml
version: '3'
services:
    jfrog-jcr:
        image: docker.bintray.io/jfrog/artifactory-jcr:latest 
        container_name: jfrog-jcr  #容器名
        restart: always
        ports:
        	- '8771:8081'  # Rest Api端口，用作上传拉取制品包
        	- '8772:8082'  # web访问端口，必须开放
        volumes:
        	- ./data:/var/opt/jfrog/artifactory  #工作目录挂载到外部服务器
```

执行命令启动 `jfrog-jcr` 容器

```bash
docker-compose up -d jfrog-jcr
```

## 2. JCR 的使用

通过 ` http://SERVER_ HOSTNAME:8772/ui/` 访问web端，输入默认用户名密码 `admin/password` ，登陆成功后即可创建 `Docker` 仓库：

![1659868523300](../blog-assets/JfrogArtifactory搭建/1659868523300.png)