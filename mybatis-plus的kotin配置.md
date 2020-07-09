最近一直在用 kotlin 写 spring，作为 kotlin 新人遇到的坑基本都能 Google 到，也就没必要在写一次了。今天将我在开发过程中使用 mybatis-plus 的 AutoGenerator kotlin 版本分享出来，给即将入坑的朋友。

通过 AutoGenerator 可以快速生成 Entity、Mapper、Mapper XML、Service、Controller 等各个模块的代码，极大的提升了开发效率。

---

## 先给`kotlin`打个广告

**如果你正在写 Java 代码，可以尝试使用 kotlin，历史的积累不会失去，kotlin 的优雅会让你爱不释手。**

## 正文

mybatis-plus 在 kotlin 中使用已经可以完美使用了,这一篇主要写 mybatis-plus 代码生成器的配置（伸手党福利），因为官方 demo 比较老了，甚至它还是 Java 代码，这是我难以接受的。

在 mybatis-plus 中使用 kotlin 模式很简单，只需

```kotlin
 gc.isKotlin = true
```

kotlin 版本完整的代码生成器

```kotlin
import com.baomidou.mybatisplus.generator.AutoGenerator
import com.baomidou.mybatisplus.generator.config.*
import com.baomidou.mybatisplus.generator.config.rules.NamingStrategy
import com.baomidou.mybatisplus.generator.engine.VelocityTemplateEngine


/**
 * @author spiderMan
 * @since 2019-09-23
 */
fun main(args: Array<String>) {

    // 表名，多个继续写
    val tableNameList = listOf("t_user", "t_user2")

    val mpg = AutoGenerator()
    // 全局配置
    val gc = GlobalConfig()
    val projectPath = System.getProperty("user.dir")
    gc.outputDir = "$projectPath/src/main/kotlin"
    gc.author = "spiderMan"
    gc.isOpen = false
    gc.isKotlin = true
    gc.isSwagger2 = true
    gc.entityName = "%sDO"
    mpg.globalConfig = gc

    // 数据源配置
    val dsc = DataSourceConfig()
    dsc.url = "jdbc:mysql://localhost:3306/t_user?serverTimezone=UTC&useUnicode=true&characterEncoding=utf-8&useSSL=true"
    dsc.driverName = "com.mysql.jdbc.Driver"
    dsc.username = "root"
    dsc.password = "zaq12wsx"
    mpg.dataSource = dsc

    // 包配置
    val pc = PackageConfig()
    // 父包名，自行修改
    pc.parent = "com.**.*"
    mpg.packageInfo = pc


    // 配置模板
    val templateConfig = TemplateConfig()

    // 配置自定义输出模板
    templateConfig.entityKt = "templates/entity.kt"
    templateConfig.mapper = "templates/mapper.kt"
    templateConfig.service = "templates/service.kt"
    templateConfig.serviceImpl = "templates/serviceImpl.kt"

    templateConfig.controller = null
    templateConfig.xml = null
    mpg.template = templateConfig

    // 策略配置
    val strategy = StrategyConfig()
    strategy.naming = NamingStrategy.underline_to_camel
    strategy.columnNaming = NamingStrategy.underline_to_camel
    strategy.superEntityClass = "com.mybatis.app.common.BaseEntity"


    // 写于父类中的公共字段
    strategy.setSuperEntityColumns("auto_id")
    strategy.setInclude(*tableNameList.toTypedArray())
    strategy.isControllerMappingHyphenStyle = true
    strategy.setTablePrefix(pc.moduleName + "_")
    mpg.strategy = strategy
    mpg.templateEngine = VelocityTemplateEngine()
    mpg.execute()
}

```

表名直接写更有效率，省去输入环节

## 模版文件

### entity.kt.vm

```v
package ${package.Entity}

#foreach($pkg in ${table.importPackages})
import ${pkg}
#end
#if(${swagger2})
import io.swagger.annotations.ApiModel
import io.swagger.annotations.ApiModelProperty
#end
/**
* <p>
* $!{table.comment}
* </p>
*
* @author ${author}
* @since ${date}
*/
#if(${table.convert})
@TableName("${table.name}")
#end
#if(${swagger2})
@ApiModel(value="${entity}对象", description="$!{table.comment}")
#end
#if(${superEntityClass})
class ${entity} : ${superEntityClass}#if(${activeRecord})<${entity}>#end() {
#elseif(${activeRecord})
class ${entity} : Model<${entity}>() {
#else
class ${entity} : Serializable {
#end

## ----------  BEGIN 字段循环遍历  ----------
#foreach($field in ${table.fields})
#if(${field.keyFlag})
#set($keyPropertyName=${field.propertyName})
#end
#if("$!field.comment" != "")
    #if(${swagger2})
    @ApiModelProperty(value = "${field.comment}")
    #else
    /**
    * ${field.comment}
    */
    #end
#end
#if(${field.keyFlag})
## 主键
#if(${field.keyIdentityFlag})
    @TableId(value = "${field.name}", type = IdType.AUTO)
#elseif(!$null.isNull(${idType}) && "$!idType" != "")
    @TableId(value = "${field.name}", type = IdType.${idType})
#elseif(${field.convert})
    @TableId("${field.name}")
#end
## 普通字段
#elseif(${field.fill})
## -----   存在字段填充设置   -----
#if(${field.convert})
    @TableField(value = "${field.name}", fill = FieldFill.${field.fill})
#else
    @TableField(fill = FieldFill.${field.fill})
#end
#elseif(${field.convert})
    @TableField("${field.name}")
#end
## 乐观锁注解
#if(${versionFieldName}==${field.name})
    @Version
#end
## 逻辑删除注解
#if(${logicDeleteFieldName}==${field.name})
    @TableLogic
#end
    #if(${field.propertyType} == "Integer")
    var ${field.propertyName}: Int? = null
    #else
    var ${field.propertyName}: ${field.propertyType}? = null
    #end
#end
## ----------  END 字段循环遍历  ----------


#if(${entityColumnConstant})
    companion object {
#foreach($field in ${table.fields})

        const val ${field.name.toUpperCase()} : String = "${field.name}"

#end
    }

#end
#if(${activeRecord})
    override fun pkVal(): Serializable? {
#if(${keyPropertyName})
        return ${keyPropertyName}
#else
        return null
#end
    }

#end
    override fun toString(): String {
        return "${entity}{" +
#foreach($field in ${table.fields})
#if($!{foreach.index}==0)
        "${field.propertyName}=" + ${field.propertyName} +
#else
        ", ${field.propertyName}=" + ${field.propertyName} +
#end
#end
        "}"
    }
}

```

### mapper.kt.vm

````v
package ${package.Mapper}

import ${package.Entity}.${entity}
import ${superMapperClassPackage}

/**
* <p>
* $!{table.comment} Mapper 接口
* </p>
*
* @author ${author}
* @since ${date}
*/
interface ${table.mapperName} : ${superMapperClass}<${entity}>

    ```

### service.kt.vm
```v
package ${package.Service}

import ${package.Entity}.${entity}
import ${superServiceClassPackage}

/**
* <p>
* $!{table.comment} 服务类
* </p>
*
* @author ${author}
* @since ${date}
*/

interface ${table.serviceName} : ${superServiceClass}<${entity}>

````

### serviceImpl.kt.vm

```v
package ${package.ServiceImpl}

import ${package.Entity}.${entity}
import ${package.Mapper}.${table.mapperName}
import ${package.Service}.${table.serviceName}
import ${superServiceImplClassPackage}
import org.springframework.stereotype.Service

/**
* <p>
* $!{table.comment} 服务实现类
* </p>
*
* @author ${author}
* @since ${date}
*/
@Service
class ${table.serviceImplName} : ${superServiceImplClass}<${table.mapperName}, ${entity}>(), ${table.serviceName}
```

### 备注

### 拒绝 XML，从我做起

- 我将配置中的生成 XML 关掉了，作为一个前端工程师出身的程序员，我实在难以接受 XML 这种丑陋语法，XML 可以做的事情注解也可以做，所以为了统一的开发体验，为了优雅，我拒绝使用它。

### 不自动生成 controller 文件

- 绝大多数情况下，表和接口名是对不上的，就不自动生成了

以上就是在我在写 kotlin 时，mybatis-plus 代码生成器全部配置，如果你需要，只要拿去随便改改就可以了，另外基础配置请自行 Google。
