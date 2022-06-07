---
title: 基于Hexo的个人博客搭建
---

> 使用 Hexo 博客框架快速搭建个人博客，配合 Github Pages 实现博客部署及域名访问

## Hexo安装

- 前置环境：安装`Node.js`、`git` 

### 配置本地`git` 仓库

 找到Git安装目录，打开Git Bash窗口，设置user.name和 user.email  

```shell
git config --global user.name niceday
git config --global user.email niceday@163.com
```

配置Git公钥

```shell
ssh-keygen -t rsa -C "niceday@163.com"
```

连按三个回车后，得到生成的公钥文件 `id_rsa.pub` ，复制其中的内容，添加到Github的`SSH KEY` 中，添加完成后，使用以下命令查看是否添加成功

```shell
ssh git@github.com
```



### Hexo搭建

Hexo 官方文档：  [文档 | Hexo](https://hexo.io/zh-cn/docs/index.html) 

进入创建好的 Blog 文件夹，右击选择 `Git Bash here` 打开命令窗口，输入以下命令

```bash
# 安装Hexo
npm install -g hexo-cli
# 检查安装是否成功
hexo -v
# 初始化hexo文件夹
hexo init blog
cd blog
# 新建一个文章
hexo new test_blog
# 生成静态页面
hexo g
# 启动服务器，本地预览 localhost:4000
hexo s
```

## Hexo 主题安装

参考 `Hexo Fluid` 官方手册：[Hexo Fluid 用户手册](https://hexo.fluid-dev.com/docs)

## Hexo 部署至 Nginx

在`Hexo` 项目目录下，执行以下命令打包 `Hexo` ，生成 `public` 目录

```bash
hexo g
```

将`public` 目录下的文件放至 nginx 的`html` 目录，启动 nginx 即可

## Github Pages 

GitHub 主页右上角加号 -> New repository ：

- Repository name 中输入 `账号名.github.io`
- 勾选 “Initialize this repository with a README”

创建仓库后，默认自动启用HTTPS，访问地址为 `https://账号名.github.io` 

### 部署 Hexo 到 Github Pages

在 Blog 文件目录下，输入以下命令

```shell
# 安装 hexo-deployer-git
npm run install hexo-deployer-git --save
```
安装完成后，修改同目录下`_config.yml`文件末尾部分

```yaml
deploy:
  type: git
  repository: git@github.com:账户名/账户名.github.io.git
  branch: main
```

完成后，运行以下命令，将Hexo部署到Github Pages，通过域名 `https://账号名.github.io` 可查看博客网站

```bash
hexo deploy
```

### Github Action 实现自动化部署 Hexo





