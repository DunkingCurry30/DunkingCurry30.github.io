---
title: docker 环境及常用镜像安装
tags: 
  - docker
  - ssh
categories: 
  - DevOps
  - 环境搭建
---
# docker环境安装

> [Docker](https://so.csdn.net/so/search?q=Docker&spm=1001.2101.3001.7020) 是一个开源的应用容器引擎，让开发者可以打包他们的应用以及依赖包到一个可移植的容器中,然后发布到任何流行的Linux机器或Windows 机器上,也可以实现虚拟化,容器是完全使用沙箱机制,相互之间不会有任何接口。 

- 参考： [Centos7安装Docker](https://blog.csdn.net/qq_26400011/article/details/113856681?ops_request_misc=%7B%22request%5Fid%22%3A%22165398523016781435481764%22%2C%22scm%22%3A%2220140713.130102334..%22%7D&request_id=165398523016781435481764&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_positive~default-1-113856681-null-null.142^v11^pc_search_result_control_group,157^v12^control&utm_term=docker安装&spm=1018.2226.3001.4187) 

- 服务器环境：CentOS 7.6

```shell
# 更新yum
yum update
# 安装yum-utils工具
yum install -y yum-utils
#安装htop
yum -y install htop
#安装tree命令
yum -y install tree
#设置阿里云镜像仓库地址
yum-config-manager \
   --add-repo \
   http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

# 更新yum软件包索引
yum makecache fase 
# 安装 docker（这里安装docker-ce版本）
yum install docker-ce docker-ce-cli containerd.io

# 配置阿里云镜像加速器
mkdir -p /etc/docker
# 配置镜像加速地址(需至阿里云平台——容器服务申请)
tee /etc/docker/daemon.json <<-'EOF'
{
"registry-mirrors": ["https://xxxxxxx.mirror.aliyuncs.com"]
}
EOF

# 重启docker
systemctl daemon-reload
systemctl restart docker

# 设置开机启动docker
systemctl enable docker


```

## docker常用命令

```shell
# 构建镜像
docker build -t 镜像名称
# 加载镜像tar包
docker load -i xxx.tar
# 删除镜像
docker rmi 镜像名称
# 保存镜像
docker save hello-world:latest -o /home/docker/images/hello-world.tar
# 启停容器
docker run -d --name 容器名称 -p 8080:80 镜像名称
docker start 镜像名称
docker stop 镜像名称
# 删除容器
docker rm 镜像名称
# 进入容器
docker exec -it 容器名称 容器命令行
# 退出容器
exit
```



# docker安装Portainer

> Portainer是一个可视化的容器镜像的图形管理工具，利用Portainer可以轻松构建，管理和维护Docker环境。 而且完全免费，基于容器化的安装方式，方便高效部署。 

```shell
# 获取portainer-ce镜像
docker search portainer
docker pull portainer/portainer-ce
# 创建volume做持久化
docker volume create portainer_data
# 启动
docker run -d -p 9000:9000 --restart=always --name=portainer \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data \
  portainer/portainer-ce

# 查看容器状态，通过 ip:9000 访问web页面
docker ps -a

# 允许远程连接docker，这里配置docker端口为2375，同步需要打开防火墙
# ！！！！注意：此方式仅适用于内网环境，生产环境请使用TLS连接，否则直接docker暴露2375端口，极易被挖矿病毒攻击
vim /usr/lib/systemd/system/docker.service
# 找到以下行，修改为
ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:2375 -H unix://var/run/docker.sock --containerd=/run/containerd/containerd.sock
# 重启docker
systemctl daemon-reload
systemctl restart docker
```

# docker开启远程安全访问（TLS）

通过`TLS`方式远程安全访问docker，这里配置docker端口为`2375`，同步需要打开防火墙

首先，创建一个ca文件夹存放公钥和私钥

```shell
mkdir /usr/local/userSh
cd /usr/local/userSh
touch tls.sh
vi tls.sh
```

将下列脚本内容复制

```shell
#!/bin/bash
#相关配置信息（除IP有用，其他基本咩有用）
#服务器IP或者域名
SERVER="服务器ip"
PASSWORD="nicetry"
COUNTRY="CN"
STATE="Sichuan"
CITY="Hangzhou"
ORGANIZATION="nicetry"
ORGANIZATIONAL_UNIT="Dev"
EMAIL="nicetry@163.com"
 
###开始生成文件###
echo "开始生成文件"
#创建密钥文件夹
mkdir -p /usr/local/ca
#切换到生产密钥的目录
cd /usr/local/ca  
#生成ca私钥(使用aes256加密)
openssl genrsa -aes256 -passout pass:$PASSWORD  -out ca-key.pem 4096
#生成ca证书，填写配置信息
openssl req -new -x509 -passin "pass:$PASSWORD" -days 3650 -key ca-key.pem -sha256 -out ca.pem -subj "/C=$COUNTRY/ST=$STATE/L=$CITY/O=$ORGANIZATION/OU=$ORGANIZATIONAL_UNIT/CN=$SERVER/emailAddress=$EMAIL"
 
#生成server证书私钥文件
openssl genrsa -out server-key.pem 4096
#生成server证书请求文件
openssl req -subj "/CN=$SERVER" -new -key server-key.pem -out server.csr
#配置白名单  你使用的是服务器Ip的话,请将前面的DNS换成IP  echo subjectAltName = IP:$SERVER,IP:0.0.0.0 >> extfile.cnf
sh -c  'echo "subjectAltName = DNS:'$SERVER',IP:0.0.0.0" >> extfile.cnf'
sh -c  'echo "extendedKeyUsage = serverAuth" >> extfile.cnf'
#使用CA证书及CA密钥以及上面的server证书请求文件进行签发，生成server自签证书
openssl x509 -req -days 3650 -in server.csr -CA ca.pem -CAkey ca-key.pem -passin "pass:$PASSWORD" -CAcreateserial  -out server-cert.pem  -extfile extfile.cnf
 
#生成client证书RSA私钥文件
openssl genrsa -out key.pem 4096
#生成client证书请求文件
openssl req -subj '/CN=client' -new -key key.pem -out client.csr
 
sh -c 'echo extendedKeyUsage=clientAuth >> extfile.cnf'
#生成client自签证书（根据上面的client私钥文件、client证书请求文件生成）
openssl x509 -req -days 3650 -in client.csr -CA ca.pem -CAkey ca-key.pem  -passin "pass:$PASSWORD" -CAcreateserial -out cert.pem  -extfile extfile.cnf
 
#更改密钥权限
chmod 0400 ca-key.pem key.pem server-key.pem
#更改密钥权限
chmod 0444 ca.pem server-cert.pem cert.pem
#删除无用文件
rm client.csr server.csr
#复制密钥文件
cp server-*.pem /etc/docker/
cp ca.pem /etc/docker/
echo "生成文件完成"
###生成结束###
```

执行脚本，在`/usr/local/ca/` 目录下生成ca证书

```shell
sh tls.sh
```

修改docker配置文件

```shell
vim /usr/lib/systemd/system/docker.service
# 找到以下行，修改为
ExecStart=/usr/bin/dockerd --tlsverify --tlscacert=/usr/local/ca/ca.pem --tlscert=/usr/local/ca/server-cert.pem --tlskey=/usr/local/ca/server-key.pem -H tcp://0.0.0.0:2375 -H unix://var/run/docker.sock --containerd=/run/containerd/containerd.sock
# 重启docker，同时需放开2375端口防火墙
systemctl daemon-reload
systemctl restart docker
```

在`Portainer`中使用`TLS` 方式，上传`cert.pem` 、`key.pem` 文件

![1654156409795](C:\Users\sherlock\AppData\Roaming\Typora\typora-user-images\1654156409795.png)

# docker安装netdata

>  Netdata 是一款 Linux 性能实时监测工具 ，其自身是一个高度优化的 Linux 守护进程，它为 Linux 系统，应用程序，SNMP 服务等提供实时的性能监测，并通过可视化图表信息呈现至web页面

```shell
# 获取netdata镜像
docker search netdata
docker pull netdata/netdata
# 启动
docker run -d --name=netdata \
  -p 19999:19999 \
  -v netdataconfig:/etc/netdata \
  -v netdatacache:/var/cache/netdata \
  -v netdatalib:/var/lib/netdata \
  -v /etc/passwd:/host/etc/passwd:ro \
  -v /etc/group:/host/etc/group:ro \
  -v /proc:/host/proc:ro \
  -v /sys:/host/sys:ro \
  -v /etc/os-release:/host/etc/os-release:ro \
  --restart unless-stopped \
  --cap-add SYS_PTRACE \
  --security-opt apparmor=unconfined \
  netdata/netdata
  
# 查看容器状态，通过 ip:19999 访问web页面
docker ps -a
```

# docker-compose安装

> docker-compose项目是[Docker](https://cloud.tencent.com/product/tke?from=10680)官方的开源项目，负责实现对Docker[容器](https://cloud.tencent.com/product/tke?from=10680)集群的快速编排。它是一个定义和运行多容器的 docker应用工具。使用compose，你能通过YMAL文件配置你自己的服务，然后通过一个命令，你能使用配置文件 创建和运行所有的服务。重点可以启动多个容器！ 

## 方式一：通过pip3安装

```shell
# 安装pip
yum -y install python-pip
# 更新pip3
pip3 install --upgrade pip
# 安装docker-compose
pip3 install docker-compose
# 查看docker-compose版本
docker-compose version
```

## 方式二：离线安装

 访问https://github.com/docker/compose/releases，下载 `docker-compose-Linux-x86_64` ，下载后重命名文件为 `docker-compose` ，可通过FTP工具上传到服务器` /usr/local/bin/ ` 目录下

```shell
# 添加可执行权限
sudo chmod +x /usr/local/bin/docker-compose
# 查看docker-compose版本
docker-compose verison
```

# docker-compose 安装 Nginx

创建`nginx` 目录，在目录下编辑 `docker-compose.yml` 文件

```bash
mkdir ./nginx && cd ./nginx
vim docker-compose.yml
```

内容如下，`version` 版本号需要与`docker-compose` 版本对应，不然会报错

CentOS7中Docker文件挂载，容器中没有执行权限，故需添加参数

```yaml
version: '3'
services:
    nginx:
        image: nginx     # 镜像名称
        container_name: nginx     # 容器名字
        restart: always     # 开机自动重启
        ports:     # 端口号绑定（宿主机:容器内）
            - '80:80'
            - '443:443'
        volumes:      # 目录映射（宿主机:容器内）
            - ./conf/nginx.conf:/etc/nginx/nginx.conf
            - ./conf.d:/etc/nginx/conf.d
            - ./html:/usr/share/nginx/html
```

需要事先准备`nginx.conf` 文件在挂载目录`./conf/nginx.conf`，否则会报试图将文件挂载至目录的错误（_因docker挂载目录文件，若挂载的宿主机目录不存在默认会创建一个新`directory` ，若原本想挂载的是 `file` 而docker默认创建的是`directory` 则会报该错误_）

- `nginx.conf`配置如下

```yaml
#user  nobody;
worker_processes  1;

events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    sendfile        on;

    keepalive_timeout  65;

    server {
        listen       80;
        server_name  localhost;

        #access_log  logs/host.access.log  main;

        location / {
            root   /usr/share/nginx/html;
            index  index.html index.htm;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
}

```

需要注意` location` 中 `root` 的配置，因为之前选择了挂载，所以需要填写为容器内的`html` 路径，否则会报`404` 错误



至此 `nginx` 目录结构为

```bash
|-- conf
|   `-- nginx.conf
|-- conf.d
|-- docker-compose.yml
`-- html
    |`-- index.html
```

执行命令启动Nginx容器，通过 `ip:80` 访问是否启动成功你

```bash
docker-compose up -d nginx
```

