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

![1654501915699](C:\Users\DunkingCurry\AppData\Roaming\Typora\typora-user-images\1654501915699.png)

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
gitlab_rails['smtp_password'] = "cR9:kRhz2-M9p4."#你的授权码
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

![1654511863688](C:\Users\DunkingCurry\AppData\Roaming\Typora\typora-user-images\1654511863688.png)

> 2. 点击进入个人设置`SSH KEYS` ，添加本地 `id_rsa.pub` 公钥（公钥生成方式可参考`《基于hexo的个人博客搭建》` 中的 `配置本地git仓库` ）

![1654512220134](C:\Users\DunkingCurry\AppData\Roaming\Typora\typora-user-images\1654512220134.png)

> 3. 本地创建一个myblog目录，右键点击` git bash` 执行以下命令

```bash
git config user.name 用户名
git config user.email 邮箱
```

> 4. 在 `GitLab` 项目页面获取 `ssh链接`，可使用 `git clone` 拉取项目，切换分支后使用 `git push` 命令推送（或使用vscode等IDE工具的插件对仓库进行操作）

![1654512603051](C:\Users\DunkingCurry\AppData\Roaming\Typora\typora-user-images\1654512603051.png)

# 

# GitLab-CI 自动化部署

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

执行`docker-compose` 命令

```bash
docker-compose up -d
#启动成功后，进入容器
docker exec -it gitlab-runner bash

# 注册 runner
gitlab-runner register
```

注册流程可参考： [ docker安装gitlab-runner自动化部署过程](https://blog.csdn.net/weixin_39934640/article/details/110132281) 

注册成功后，可在`GitLab—设置—CI/CD` 中查看 `runner` 状态是否可用

![](../../blog-assets/Gitlab搭建及实现CI/20220607095216.png)

