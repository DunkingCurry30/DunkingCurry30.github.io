---
title: docker 部署 gitlab gitlab-runner 实现 CI
tags: 
  - docker
  - gitlab
  - ssh
categories: 
  - DevOps
  - 环境搭建
---
# GitLab私服搭建

GitLab 是一个用于仓库管理系统的开源项目，使用[Git](https://baike.baidu.com/item/Git)作为代码管理工具，并在此基础上搭建起来的Web服务 

> 参考官方Doc：  [GitLab Docker images | GitLab](https://docs.gitlab.com/ee/install/docker.html) 

```bash
#创建宿主机挂载目录
mkdir -p /data/docker/gitlab/{config,data,logs}
```

在`/data/docker/gitlab/` 目录下创建 `docker-compose.yml` 文件，内容如下

```yaml
version: '3.6'
services:
  gitlab:
    image: 'twang2218/gitlab-ce-zh' #这里使用twang团队的汉化社区版
    container_name: gitlab # 生成的docker容器的名字
    restart: always
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        external_url 'http://ip:8998' # 此处填写所在服务器ip若有域名可以写域名
        gitlab_rails['gitlab_shell_ssh_port'] = 2224
    ports:
      - '8998:8998' # 此处端口号须与 external_url 中保持一致，左边和右边都要一样
      - '2224:22' # 这里的2224和上面的2224一致，但是右边必须是22，不能是其他
    volumes:     # 目录映射（宿主机:容器内）
        - ./data:/var/opt/gitlab
        - ./logs:/var/log/gitlab
        - ./config:/etc/gitlab
    shm_size: '256m'
```

执行`docker-compose` 命令，进行安装和容器启动，开通端口`8998` 和 `2224` 的防火墙

```bash
docker-compose up -d gitlab
```

启动成功后，`ip:8998` 访问web端进行 Gitlab 设置新密码，使用`root` 账号登录进行使用

![1654501915699](../blog-assets/Gitlab搭建及实现CI/1654501915699.png)

## GitLab邮箱配置

> 前置条件：准备一个邮箱（以网易举例，需要在网易邮箱设置中开启 POP3/SMTP 服务，获得授权码）

```bash
vim /data/docker/gitlab/config/gitlab.rb
```

修改`gitlab.rb` 的以下配置

```bash
### GitLab email server settings
gitlab_rails['smtp_enable'] = true
gitlab_rails['smtp_address'] = "smtp.163.com"#网易邮箱发送服务器
gitlab_rails['smtp_port'] = 465
gitlab_rails['smtp_user_name'] = "dunkingcurry30@163.com"#你的邮箱地址
gitlab_rails['smtp_password'] = "xxxxxxxxx"#你的授权码
gitlab_rails['smtp_domain'] = "smtp.163.com"
gitlab_rails['smtp_authentication'] = "login"
gitlab_rails['smtp_enable_starttls_auto'] = true
gitlab_rails['smtp_tls'] = true

### Email Settings
gitlab_rails['gitlab_email_enabled'] = true
gitlab_rails['gitlab_email_from'] = 'dunkingcurry30@163.com'#你的邮箱地址
gitlab_rails['gitlab_email_display_name'] = 'GitLab'
```

配置后，请删除镜像后重新用`docker-compose` 命令启动容器，启动成功后

```bash
#通过portainer进入gitlab容器或者使用docker命令都可
docker exec -it gitlab /bin/bash
#进入gitlab控制台
gitlab-rails console
#检查邮箱发送方式是否为smtp
ActionMailer::Base.delivery_method
#检查邮箱信息是否为设置的值
ActionMailer::Base.smtp_settings

#发送测试邮件，查看是否收到
Notify.test_email('157468143@qq.com','Hello World', 'This is a test message').deliver_now
```



# GitLab创建和拉取项目

> 1. 登录`GitLab` 创建项目 `myblog`  ，建议勾选创建 `README.md` ，创建完成后建议再新建一个`dev` 分支（因`GitLab` 默认 `master` 分支受保护，只能拉取无法推送）

![1654511863688](../blog-assets/Gitlab搭建及实现CI/1654511863688.png)

> 2. 点击进入个人设置`SSH KEYS` ，添加本地 `id_rsa.pub` 公钥（公钥生成方式可参考`《基于hexo的个人博客搭建》` 中的 `配置本地git仓库` ）

![1654512220134](../blog-assets/Gitlab搭建及实现CI/1654512220134.png)

> 3. 本地创建一个myblog目录，右键点击` git bash` 执行以下命令

```bash
git config user.name 用户名
git config user.email 邮箱
```

> 4. 在 `GitLab` 项目页面获取 `ssh链接`，可使用 `git clone` 拉取项目，切换分支后使用 `git push` 命令推送（或使用vscode等IDE工具的插件对仓库进行操作）

![1654512603051](../blog-assets/Gitlab搭建及实现CI/1654512603051.png)



# GitLab-CI 自动化部署

> **1. 安装及注册 gitlab-runner**

首先，安装 `gitlab-runner` 

```bash
# 创建宿主机挂载目录
mkdir -p /data/docker/gitlab/gitlab-runner/config
```

在创建的`config` 目录下创建文件`config.toml` ，内容如下

```bash
concurrent = 1
check_interval = 0

[session_server]
  session_timeout = 1800
```

在 GitLab 的 `docker-compose.yml` 中增加下列内容，注意缩进

```bash
    gitlab-runner:
        image: 'gitlab/gitlab-runner'
        container_name: gitlab-runner
        restart: always
        volumes:
            - ./gitlab-runner/config:/etc/gitlab-runner 
```

执行`docker-compose` 命令，容器启动成功后注册 `runner` ，

其中，注册流程可参考： [ docker安装gitlab-runner自动化部署过程](https://blog.csdn.net/weixin_39934640/article/details/110132281) 

```bash
docker-compose up -d
#启动成功后，进入容器
docker exec -it gitlab-runner bash

# 注册 runner
gitlab-runner register
```

注册成功后，可在`GitLab—设置—CI/CD` 中查看 `runner` 状态是否可用

![20220607095216](../blog-assets/Gitlab搭建及实现CI/20220607095216.png)



> **2. 在项目根目录下编写`.gitlab-ci.yml` 脚本** 

内容如下（第一次写，看不明白没关系，弄懂 `stage` ，`job` 的概念就行），后续会详细解析该脚本的配置

```yaml
## stages：通俗讲：就是将项目，从打包部署，分几个作业job，比如这里就分两个步骤，一个 build,打包编译， deploy：部署
## 在每个作业执行之前，有一些通用的执行操作，例如 SSH 认证配置、缓存等，放在 before_secript、cache 中
# image: node # 就是所依赖的环境 在 gitrunner-cli注册的时候，已经写了此处可以省略不写
stages:
    - build    
    - deploy
     
before_script:
  - echo "gitlab-ci start..."
  - eval `ssh-agent -s` # 设置 SSH 免密登录
  - ssh-add <(echo "$SSH_KEY")
  - mkdir -p ~/.ssh
  - chmod 700 ~/.ssh
  - ssh-keyscan 部署服务器IP > ~/.ssh/known_hosts
  - chmod 644 ~/.ssh/known_hosts 
build_job:
  
  stage: build  # 这一个stage：的值就是上面的 stages下的第一个 build
  tags: # tag 在注册的时候，有提到过，分两步骤，第一个是 buil,所以这里也对应上，注册的时候是自定义的名称
  - build
  script: # 这个就是操作命令 或者写脚本路径
  - npm install -g cnpm --registry=https://registry.npm.taobao.org #切换至cnpm，提升速度
  - echo `pwd` # 输出当前路径
  - cnpm install -g hexo-cli  # 安装 hexo-cli
  - cnpm install 
  - hexo clean && hexo g # 打包生成 public 文件夹
 
# 缓存，因为是容器执行，每次执行的时候都会重新打包，缓存起来就不需要再次安装一次  
cache:
  paths:
   - node_modules/
   - public/  

# 第二个任务 就是专业词语 job ,第一个是先 build ,这个跟上面的build一样名称自定义
deploy_job:
     stage: deploy  # 这一个stage：的值就是上面的 stages下的第一个 deploy
     tags:
       - deploy # tag 在注册的时候，有提到过，分两步骤，第二个个是 deploy,所以这里也对应上，注册的时候是自定义的名称
     script:
      - pwd #  也输出下路径
      - mv public html
      - ls
      - scp -r ./html/ root@部署服务器IP:/opt/docker/nginx  # 将打包好的文件上到发布项目的服务器中的。放到nginx能访问到的文件夹下
      - echo "gitlab-ci success."
```



脚本编写完成后，可以将内容放到 `GitLab` 进行语法检测，看能否正确识别出各个步骤操作

![1654581145899](../blog-assets/Gitlab搭建及实现CI/1654581145899.png)



> **3. 设置SSH免密登录**

上述脚本中使用了一个变量 `SSH_KEY` ，该变量为 `gitlab-runner` 服务器的 `ssh私钥` 信息。

这里简单解释一下，我们知道 `git-runner` 要使用 SSH 登录项目部署的服务器，需要创建一对公私钥，将其公钥添加到 `部署服务器` 的 ssh 公钥配置中，登录时再通过私钥去做验证

- 参考： [Linux - 配置SSH免密登入](https://blog.csdn.net/weixin_46232508/article/details/106397322) 

因此我们在 `gitlab-runner` 安装的服务器上，输入如下命令

```bash
# 生成公私钥
ssh-keygen -t rsa -C "niceday@163.com"
```

- 将 `~/.ssh` 目录下的公钥 `id_rsa.pub` 的内容添加至 部署项目的服务器 `~/.ssh/anthorized_keys` 中
- 将目录下私钥 `id_rsa` 的内容添加至变量 `SSH_KEY` 中

![1654583291460](../blog-assets/Gitlab搭建及实现CI/1654583291460.png)



> **4. push 脚本到 Gitlab**

验证无误后，就将 `.gitlab-ci.yml` 脚本 push 到远程仓库，在`GitLab—项目—CI/CD` 下就可以查看自动化作业执行的情况了

![1654581464922](../blog-assets/Gitlab搭建及实现CI/1654581464922.png)



点击每个作业步，也可以查看具体的命令执行信息

![1654581569487](../blog-assets/Gitlab搭建及实现CI/1654581569487.png)



> **5. 验证执行结果**

流水线显示执行成功后，可通过浏览器验证内容是否更新，也可上服务器看文件是否更新

![1654584003746](../blog-assets/Gitlab搭建及实现CI/1654584003746.png)

**至此，大功告成！**



# 将 GitLab 提交同步至 Github

- 参考： [GitLab和GitHub的双向同步](https://www.cnblogs.com/sxdcgaq8080/p/10530176.html) 
- 提高 `Github` 访问速度： [github域名解析优化](https://blog.csdn.net/wplblog/article/details/120967043?utm_medium=distribute.pc_feed_404.none-task-blog-2~default~BlogCommendFromBaidu~Rate-1-120967043-blog-null.pc_404_mixedpudn&depth_1-utm_source=distribute.pc_feed_404.none-task-blog-2~default~BlogCommendFromBaidu~Rate-1-120967043-blog-null.pc_404_mixedpud) 

> 1. 在 `github` 上创建一个 `token` 

登录 Github，找到 `settings - Developer settings - Personal access tokens` 设置，创建包含`repo` 权限的新`token` ，生成的 `token` 记录下来，网站不会保存

![1654590060334](../blog-assets/Gitlab搭建及实现CI/1654590060334.png)



> 2. 获取要同步的 Github 库

在 Github 上找到要同步的项目库，获取版本库链接

![1654590334074](../blog-assets/Gitlab搭建及实现CI/1654590334074.png)



将该链接内容改动如下，在`github.com`前拼接 `用户名:步骤1的token@` 

```http
https://你的用户名:你的token@github.com/DunkingCurry30/DunkingCurry30.github.io.git
```



> 3. Gitlab配置Github库

登录 Gitlab，选择要同步的项目 `设置 - 仓库 - 推送到远程仓库 ` ，填写上面得到的链接后保存修改

![1654591468606](../blog-assets/Gitlab搭建及实现CI/1654591468606.png)



点击 `现在更新` 立即进行同步，更新成功后查看 github，成功同步

![1654591646389](../blog-assets/Gitlab搭建及实现CI/1654591646389.png)



> 至此，结合 [基于Hexo的个人博客搭建](/2022/06/07/基于Github个人博客搭建/) 可实现通过本地提交代码至 `Gitlab` 同时触发以下自动化流程 
>
> - `Gitlab CI` 自动部署至云服务器
> - `Gitlab` 提交同步 `Github` 自动部署至 `Github Pages` 