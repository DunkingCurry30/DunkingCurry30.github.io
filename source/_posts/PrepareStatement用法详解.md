---
title: PrepareStatement用法
date: 2022/7/1 20:00:00
tags: 
  - java
  - mysql
categories: 
  - Java后端
  - 数据库
---

# PrepareStatement 基本用法

## 1. 加载驱动

首先在`pom.xml` 中引入 `mysql` 依赖

```xml
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.23</version>
</dependency>
```

通过以下代码加载驱动

```java
Class.forName("com.mysql.cj.jdbc.Driver");
```



## 2. 获取 Connection 对象

数据库连接需要数据库链接、用户名、密码三个参数，建议配置在`application.properties` 中，方便统一管理（当然也可以直接在使用的地方用字符串变量）

```properties
#PrepareStatement
#数据库链接
pstmt.dbUrl="jdbc:mysql://localhost:3306/mydb?serverTimezone=UTC&characterEncoding=utf8&useUnicode=true"
#用户名
pstmt.dbUser=root
#密码
pstmt.dbPwd=123456
```

通过以下代码获取connection

```java
Connection connection = DriverManager.getConnection(dbUrl,dbUser,dbPwd);
```



## 3. 获取 PrepareStatement 对象

根据需求创建Sql语句字符串，在需要加入参数的地方用 `?` 占位符代替

```java
//查询符合指定年龄和性别的员工姓名
String sql = "SELECT a.name FROM staff_info a WHERE a.age = ? and a.sex = ?";
```

接下来，创建 `PrepareStatement`对象对sql进行预编译

```java
PreparedStatement pstmt = connection.prepareStatement(sql);
```



## 4. 设置参数

`PrepareStatement`对象提供多种参数类型方法，**使用时依据数据库字段类型一一对应**，设置时需注意第一个参数为占位符`?` 的索引且**从 1 开始**，例如之前sql的设置应为

```java
pstmt.setInt(1,35);
pstmt.setString(2,"男");
```



## 5. SQL执行

### 单条语句执行

```java
//单次执行
pstmt.execute();
```

### 批量执行

```java
//第一批
pstmt.setInt(1,35);
pstmt.setString(2,"男");
//加入批次
pstmt.addBatch();

//第二批
pstmt.setInt(1,24);
pstmt.setString(2,"女");
pstmt.addBatch();

//统一批量执行
pstmt.executeBatch();
```



## 6. 完整示例

```java
@Component
public class PrepareStatementUtils {
    private  final Logger LOGGER = LoggerFactory.getLogger(PrepareStatementUtils.class);

    @Value("${pstmt.dbUrl}")
    private  String dbUrl;

    @Value("${pstmt.dbUser}")
    private  String dbUser;

    @Value("${pstmt.dbPwd}")
    private  String dbPwd;

    public void prepareStatementExecute() {
        try {
            //加载 mysql 驱动
            Class.forName("com.mysql.cj.jdbc.Driver");
            //获取链接
            Connection connection = DriverManager.getConnection(dbUrl,dbUser,dbPwd);
            //获取preparestatement对象
            String sql = "SELECT a.name FROM staff_info a WHERE a.age = ? and a.sex = ?";
            PreparedStatement pstmt = connection.prepareStatement(sql);

            //单次执行
            pstmt.setInt(1,35);
            pstmt.setString(2,"男");
            pstmt.execute();

            //批量执行
            //第一批
            pstmt.setInt(1,35);
            pstmt.setString(2,"男");
            //加入批次
            pstmt.addBatch();

            //第二批
            pstmt.setInt(1,24);
            pstmt.setString(2,"女");
            pstmt.addBatch();

            //统一执行
            pstmt.executeBatch();

        } catch (ClassNotFoundException | SQLException e) {
            LOGGER.error(e.getMessage());
        }
    }
}
```



# PrepareStatement 防SQL注入

PrepareStatement 可以有效防止 `sql注入` ，其方式是把用户非法输入的单引号用反斜杠`\` 做了转义

# 