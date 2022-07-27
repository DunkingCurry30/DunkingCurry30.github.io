---
title: devops需求问题汇总
date: 2022/7/16 20:00:00
tags: 
  - java
categories: 
  - Java后端
---

> 参考： [(17条消息) java中属性命名get字母大小写问题_ammafree的博客-CSDN博客](https://blog.csdn.net/u010744399/article/details/52523126) 

1. datax 同步多数据源数据库
2. crontab设置定时任务，实现datax 每日定时同步
3. java中的定时任务 @Schedule
4. java中的多线程查询后，汇总返回（countdownlatch）： [java效率之异步查询汇总数据_weixin_43030390的博客-CSDN博客_java异步查询数据库](https://blog.csdn.net/weixin_43030390/article/details/86015523?ops_request_misc=%7B%22request%5Fid%22%3A%22165874492716782395392770%22%2C%22scm%22%3A%2220140713.130102334..%22%7D&request_id=165874492716782395392770&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduend~default-1-86015523-null-null.142^v33^new_blog_pos_by_title,185^v2^control&utm_term=异步查询数据库%2C汇总&spm=1018.2226.3001.4187) 
5. 多线程执行时，变量赋值的问题
6. 线程池应保证单例，不能在方法里new
7. 涉及group by的慢查询： [【mysql】group by 特别慢，优化方法_乱七八糟的兔子的博客-CSDN博客](https://blog.csdn.net/qq_31949853/article/details/84984305) 
8. 数据库大表，索引和分区的建立