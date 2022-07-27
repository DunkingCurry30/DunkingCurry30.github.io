---
title: Linux中的定时任务crontab
date: 2022/7/28 20:00:00
tags: 
  - shell
categories: 
  - Linux
---

# 一、crontab 功能

通过 `crontab` 命令，我们可以在固定的间隔时间执行指定的系统指令或 `shell` 脚本。时间间隔的单位可以是分钟、小时、日、月、周及以上的任意组合。这个命令非常适合周期性的日志分析或数据备份等工作。



# 二、crontab 安装

```bash
## 安装crontab
yum install crontabs

## 启动服务
service crond start    
## 关闭服务
service crond stop    
## 重启服务
service crond restart   
## 重新载入配置
service crond reload
## 查看crontab服务状态：
service crond status
## 手动启动crontab服务：
service crond start
## 查看crontab服务是否已设置为开机启动，执行命令：
chkconfig --list
## 加入开机自动启动：
chkconfig --level 35 crond on
```



# 三、crontab 命令

命令格式：

```bash
crontab [-u user] file
crontab [-u user] [-e|-l|-r]
```

参数说明：

- `-u user`：用来设定某个用户的crontab服务，例如，“-u ixdba”表示设定ixdba用户的crontab服务，此参数默认由root用户来运行；
- `file`：file是命令文件的名字，表示将文件 `file` 做为crontab的任务列表文件并载入crontab；
- `-e`：编辑某个用户的crontab文件内容。如果不指定用户，则表示编辑当前用户的crontab文件；
- `-l`：显示某个用户的crontab文件内容，如果不指定用户，则表示显示当前用户的crontab文件内容；
- `-r`：删除定时任务配置，从/var/spool/cron目录中删除某个用户的crontab文件，如果不指定用户，则默认删除当前用户的crontab文件；
- `-i`：在删除用户的crontab文件时给确认提示。

高频常用：

```bash
crontab -l [-u user]            ## 列出用户目前的crontab.
crontab -e [-u user]            ## 编辑用户目前的crontab.
```



# 四、调度配置

## 1. 基本格式

```bash
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
  *  *  *  *  * user-name command to be executed
```

即从左至右依次为 `分 时 日 月 周 [用户] 命令` ：

- 第1列表示分钟1～59 每分钟用*或者 */1表示

- 第2列表示小时0～23（0表示0点） 7-9表示：8点到10点之间

- 第3列表示日期1～31

- 第4列表示月份1～12

- 第5列标识号星期0～6（0表示星期天）
- 第6列标识执行用户（非必需）

- 第7列要运行的命令或 `shell` 脚本

> **【扩展】**相信你也发现了，`crontab` 最小只支持到分钟级别的任务执行，如果想实现秒级任务执行，我们需要借助 `cron表达式` ，该表达式使用非常广泛，在spring框架中也常用来设置定时任务的执行频率，典型应用就是 `@Schedule(cron)` 注解

## 2. 一些配置示例

```bash
*/1 * * * * date >> /root/date.txt

# 上面的例子表示每分钟执行一次date命令

30 21 * * * /usr/local/etc/rc.d/httpd restart

# 上面的例子表示每晚的21:30重启apache。

45 4 1,10,22 * * /usr/local/etc/rc.d/httpd restart

# 上面的例子表示每月1、10、22日的4 : 45重启apache。

10 1 * * 6,0 /usr/local/etc/rc.d/httpd restart

# 上面的例子表示每周六、周日的1 : 10重启apache。

0,30 18-23 * * * /usr/local/etc/rc.d/httpd restart

# 上面的例子表示在每天18 : 00至23 : 00之间每隔30分钟重启apache。

0 23 * * 6 /usr/local/etc/rc.d/httpd restart

# 上面的例子表示每星期六的11 : 00 pm重启apache。

* */1 * * * /usr/local/etc/rc.d/httpd restart

# 上面的例子每一小时重启apache

* 23-7/1 * * * /usr/local/etc/rc.d/httpd restart

# 上面的例子晚上11点到早上7点之间，每隔一小时重启apache

0 11 4 * mon-wed /usr/local/etc/rc.d/httpd restart

# 上面的例子每月的4号与每周一到周三的11点重启apache

0 4 1 jan * /usr/local/etc/rc.d/httpd restart

# 上面的例子一月一号的4点重启apache
```



# 五、crontab 日志查看

 `linux` 中默认情况下，`crontab` 中执行的日志写在 `/var/log` 下 ；

```bash
[root@VM-12-14-centos]#ls /var/log/cron*
/var/log/cron /var/log/cron.1 /var/log/cron.2 /var/log/cron.3 /var/log/cron.4
```

`crontab` 的日志比较简单，当然也可以将每条 `crontab` 中的任务增加自己的日志，有利于查找执行失败原因；

```bash
0 6 * * * /opt/script/test.sh >> /opt/script/log/mylog.log 2>&1                //把错误输出和标准输出都输出到mylog.log中
```

因此查看 `crontab` 日志可以直接查看 `log` 文件信息

```bash
tail -f /var/log/cron       //查看最新的日志
tail -100 /var/log/cron     //查看最新的100条日志
```



> 参考文章： [Linux crontab配置](https://www.cnblogs.com/Transkai/p/10440904.html) 



