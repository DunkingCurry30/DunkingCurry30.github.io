---
title: Mybatis Plus提升CRUD效率（二）
date: 2023/07/09 20:00:00
tags: 
  - CRUD
categories: 
  - Java后端
  - 数据库
---
# 一、MP 自动填充

> 参考:  [自动填充功能 | MyBatis-Plus (baomidou.com)](https://baomidou.com/pages/4c6bcf/)

## 应用场景

对于项目中的各个实体都具备公共字段，过去我们都是分别维护，如今可通过自动填充让业务实体只关注自身业务字段，无需处理公共字段的填充，由此可带来以下优势：

- 统一实体对象管理，简化实体结构
- 由MP统一处理，便捷的进行CRUD、主键生成及逻辑删除等操作

首先定义一个基础实体类BaseBO，里面包含主键、业务主键、创建人、创建时间等公共字段：

```java
@Data
public class BaseDO implements Serializable {

    /**
     * 主键ID
     */
    @ApiModelProperty("主键ID")
    @TableId(value = "id", type = IdType.AUTO)
    private Long id;

    /**
     * 业务主键
     */
    @ApiModelProperty("业务主键")
    @TableField(value = "bid", fill = FieldFill.INSERT)
    private Long bid;

    /**
     * 创建时间
     */
    @ApiModelProperty("创建时间")
    @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss")
    @TableField(value = "create_at", fill = FieldFill.INSERT)
    private Date createAt;

    /**
     * 创建人
     */
    @ApiModelProperty("创建人")
    @TableField(value = "create_by", fill = FieldFill.INSERT)
    private String createBy;

    /**
     * 更新时间
     */
    @ApiModelProperty("更新时间")
    @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss")
    @TableField(value = "update_at", fill = FieldFill.INSERT_UPDATE)
    private Date updateAt;

    /**
     * 更新人
     */
    @ApiModelProperty("更新人")
    @TableField(value = "update_by", fill = FieldFill.INSERT_UPDATE)
    private String updateBy;

    /**
     * 逻辑删除 0-未删除 1-已删除
     */
    @ApiModelProperty("逻辑删除")
    @TableField(value = "deleted", fill = FieldFill.INSERT)
    @TableLogic(value = "0", delval = "1")
    private Integer deleted;
  
    @ApiModelProperty(hidden = true)
    @TableField(exist = false)
    private static final long serialVersionUID = 1L;
}
```

对其中的每个字段我们需要实现对应的自动填充功能，如创建人从请求头填入的`ThreadLocal`获取填充、创建时间及更新时间生成、业务主键生成等，而利用MP我们可以轻松做到这一点。

## 填充原理

- 实现元对象处理器接口：com.baomidou.mybatisplus.core.handlers.MetaObjectHandler

## 业务实践

- 注解填充字段 `@TableField(.. fill = FieldFill.INSERT)`

其中枚举 `FieldFill` 可选值如下：

```java
public enum FieldFill {
    /**
     * 默认不处理
     */
    DEFAULT,
    /**
     * 插入填充字段
     */
    INSERT,
    /**
     * 更新填充字段
     */
    UPDATE,
    /**
     * 插入和更新填充字段
     */
    INSERT_UPDATE
}
```

- 自定义实现类 MyMetaObjectHandler

```java
@Component
@Slf4j
public class MyBaseMetaObjectHandler implements MetaObjectHandler {

    @SneakyThrows
    @Override
    public void insertFill(MetaObject metaObject) {
        // 设置基础属性为空则自动填充
        Object bid = getFieldValByName("bid", metaObject);
        if(ObjectUtils.isEmpty(bid)){
            setFieldValByName("bid",SequenceUtils.nextId(),metaObject);
        }
        Object createAt = getFieldValByName("createAt", metaObject);
        if(ObjectUtils.isEmpty(createAt)){
            setFieldValByName("createAt",new Date(),metaObject);
        }
        Object createBy = getFieldValByName("createBy", metaObject);
        if(ObjectUtils.isEmpty(createBy)){
            setFieldValByName("createBy",UserThreadLocal.getUserIdWithName(),metaObject);
        }
        Object updateAt = getFieldValByName("updateAt", metaObject);
        if(ObjectUtils.isEmpty(updateAt)){
            metaObject.setValue("updateAt", new Date());;
        }
        Object updateBy = getFieldValByName("updateBy", metaObject);
        if(ObjectUtils.isEmpty(updateAt)){
            metaObject.setValue("updateBy", UserThreadLocal.getUserIdWithName());;
        }
        Object deleted = getFieldValByName("deleted", metaObject);
        if(ObjectUtils.isEmpty(deleted)){
            setFieldValByName("deleted",0,metaObject);
        }
        Object version = getFieldValByName("version", metaObject);
        if(ObjectUtils.isEmpty(version)){
            version = SequenceUtils.nextId();
            setFieldValByName("version",version,metaObject);
        }
    }

    @Override
    public void updateFill(MetaObject metaObject) {
        log.info("公共字段自动填充【update】");
        Object updateAt = getFieldValByName("updateAt", metaObject);
        if(ObjectUtils.isEmpty(updateAt)){
            metaObject.setValue("updateAt", new Date());;
        }
        Object updateBy = getFieldValByName("updateBy", metaObject);
        if(ObjectUtils.isEmpty(updateAt)){
            metaObject.setValue("updateBy", UserThreadLocal.getUserIdWithName());;
        }
    }
}
```

至此，我们再使用业务实体继承BaseDO，使业务实体只需关注自身业务字段，无需处理公共字段，简单示例如下：

```java
/**
 * 餐品信息维度表
 * @author DunkingCurry
 * @TableName dim_busi_order_info
 */
@EqualsAndHashCode(callSuper = true)
@TableName(value ="dim_busi_order_info")
@Data
public class DimBusiOrderInfo extends BaseDO {

    /**
     * 餐品种类
     */
    @TableField(value = "meal_category")
    private String mealCategory;

    /**
     * 餐品中文名
     */
    @TableField(value = "meal_name_cn")
    private String mealNameCn;

    /**
     * 餐品英文名
     */
    @TableField(value = "meal_name_en")
    private String mealNameEn;

    /**
     * 餐品单价
     */
    @TableField(value = "meal_unit_price")
    private BigDecimal mealUnitPrice;

}
```

# 二、MP 代码生成器

> 适用版本：mybatis-plus-generator 3.5.1 及其以上版本

## 配置准备

- 基础Entity类 BaseDO
- 自动填充实现类 MyMetaObjectHandler
- 以下依赖引入

```xml
<!-- Mybatis-plus-join 依赖  -->
<dependency>
    <groupId>com.github.yulichang</groupId>
    <artifactId>mybatis-plus-join-boot-starter</artifactId>
    <version>1.4.5</version>
</dependency>
<!-- Mybatis-plus generator-->
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-generator</artifactId>
    <version>3.5.1</version>
</dependency>
<!-- free maker 模板-->
<dependency>
    <groupId>org.freemarker</groupId>
    <artifactId>freemarker</artifactId>
</dependency>
```

> 注意，此处使用了 Mybatis-plus-join 做联表增强，若只使用MP功能可不引入，对应在下述配置中修改对应BaseMapper、BaseService即可

## 自定义代码生成器实现

具体代码实现如下：

```java
/**
 * Mybatis Plus 代码生成器
 * @author : DunkingCurry
 * @createTime : 15/7/2023 下午12:42
 */
public class MpFastAutoGenerator {

    /**
     * 执行 run
     */
    public static void main(String[] args){

        // 需要生成代码的数据表清单, 英文逗号,分隔
        String tableNames = "test_table_template";

        // 数据源配置
        String url = "jdbc:mysql://127.0.0.1:3306/devdb?serverTimezone=UTC&characterEncoding=utf8&useUnicode=true";
        String username = "root";
        String password = "root";
        String projectPath = System.getProperty("user.dir");

        FastAutoGenerator
                // 数据源配置
                .create(url, username, password)
                // 全局配置
                .globalConfig(builder -> builder.fileOverride()
                        .outputDir(projectPath + "/src/main/java")
                        .disableOpenDir()
                        .author("DunkingCurry")
                        .enableSwagger()
                        .dateType(DateType.ONLY_DATE)
                        .commentDate("yyyy-MM-dd"))
                // 包配置
                .packageConfig(builder -> builder.parent("com.learn.sc.springbootdemo")
                        .entity("dao.entity")
                        .service("service")
                        .serviceImpl("service.impl")
                        .mapper("dao.mapper")
                        .controller("web.controller")
                        // 自定义Mapper XML位置至resource目录下
                        .pathInfo(Collections.singletonMap(OutputFile.mapperXml, projectPath + "/src/main/resources/mapper/default")))
                // 策略配置
                .strategyConfig(builder -> builder
                        // -- 生成表清单 --
                        .addInclude(tableNames.trim().split(","))
                        // -- Entity 策略配置 --
                        .entityBuilder()
                        // 设置entity父类
                        .superClass(BaseDO.class)
                        // 设置父类公共字段
                        .addSuperEntityColumns("id", "bid", "created_by", "created_at", "updated_by", "updated_at", "deleted", "version")
                        .formatFileName("%sDO")
                        .disableSerialVersionUID()
                        .enableChainModel()
                        .enableLombok()
                        .enableTableFieldAnnotation()
                        .versionColumnName("version")
                        // 逻辑删除字段
                        .logicDeleteColumnName("deleted")
                        // 设置字段映射为下划线转驼峰
                        .naming(NamingStrategy.underline_to_camel)
                        .columnNaming(NamingStrategy.underline_to_camel)
                        // -- Controller --
                        .controllerBuilder()
                        .enableHyphenStyle()
                        .enableRestStyle()
                        .formatFileName("%sController")
                        // -- Service 策略配置 --
                        .serviceBuilder()
                        .superServiceClass(MPJBaseService.class)
                        .superServiceImplClass(MPJBaseServiceImpl.class)
                        .formatServiceFileName("%sService")
                        .formatServiceImplFileName("%sServiceImpl")
                        // -- Mapper 策略配置 --
                        .mapperBuilder()
                        .superClass(MPJBaseMapper.class)
                        .enableMapperAnnotation()
                        .enableBaseResultMap()
                        .enableBaseColumnList()
                        .formatMapperFileName("%sMapper")
                        .formatXmlFileName("%sMapper"))
            	// 使用freemaker模板引擎, 需引入对应依赖
                .templateEngine(new FreemarkerTemplateEngine())
                // 执行
                .execute();
    }
}

```

> 注：以上包含配置可参阅 [代码生成器配置新 | MyBatis-Plus](https://baomidou.com/pages/981406/#数据库配置-datasourceconfig) 进行自定义
