Spring Data JPA 是 Spring Boot 体系中约定优于配置的最佳实现，大大简化了项目中数据库的操作。国内对接纳度不是很高，所以相关资源也比较少，这一篇将结合实际业务场景讲解 jpa 的使用方法，只谈实际业务中的使用，不做具体代码研究。

---

- Spring Boot 版本 2.2.2
- jdk 版本 1.8
- kotlin 版本 1.3.61

所以代码将以 kotlin 实现，不熟悉 kotlin 请点击[传送门](https://www.kotlincn.net/docs/reference/)，半个小时让你掌握一门优雅的新语言，用 kotlin 写 spring 生活质量都提高了。

## 快速开始

### 添加依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
 <dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
```

### 添加配置文件

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/test?serverTimezone=GMT%2b8&useUnicode=true&characterEncoding=utf-8&useSSL=true
spring.datasource.username=root
spring.datasource.password=123456
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

spring.jpa.properties.hibernate.hbm2ddl.auto=validate
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL5InnoDBDialect
#SQL 输出
spring.jpa.show-sql=true
#format 一下 SQL 进行输出
spring.jpa.properties.hibernate.format_sql=true
```

**重点是时区 GMT+8 的设置：serverTimezone=GMT%2b8**

hibernate.hbm2ddl.auto 参数

- create 每次加载 Hibernate 都会删除旧的表，生成新的表
- create-drop 每次加载 Hibernate 都会生成新的表， sessionFactory 一关闭，表就自动删除。
- update **最常用**，每次根据 model 自动更新表结构
- validate **作者推荐** 验证创建数据库表结构，不创建表和更新表结构，**你的领导不会信任你的，创建表，加减字段还是自己操作 sql 更放心**。

### 实体类

#### BaseEntity 基础实体

```kotlin
@MappedSuperclass
@EntityListeners(AuditingEntityListener::class)
open class BaseEntity(
        //自增ID
        @Id
        @GeneratedValue(strategy = GenerationType.IDENTITY)
        @Column(name = "auto_id")
        var autoId: Long? = null,
        // 表记录状态：0 正常 1 逻辑删除
        var status: Byte? = null,
        // 创建时间
        @CreationTimestamp
        @Column(name = "create_time")
        var createTime: LocalDateTime? = null,
        // 最后修改时间
        @UpdateTimestamp
        @Column(name = "last_modify_time")
        var lastModifyTime: LocalDateTime? = null
)
```

在实际业务场景中，每个表都会有固定的公共字段，我们不能每个实体类都写上这些字段是很蠢的，所以要将公共字段写在 BaseEntity 中，其他实体类继承。

- `@MappedSuperclass` 父类必加注解，不加会报错
- `@EntityListeners(AuditingEntityListener::class)` 不加此注解的话`@CreatedDate`,`@LastModifiedDate`不起作用
- `@CreationTimestamp` 自动添加创建时间
- `@UpdateTimestamp` 自动添加修改时间
- `@Id` 主键
- `@GeneratedValue(strategy = GenerationType.IDENTITY)` strategy: 表示主键生成策略,有 AUTO、INDENTITY、SEQUENCE 和 TABLE 4 种，`IDENTITY`:数据库自己生成，我比较常用，其他用法自行 google
- `@Column(name = "auto_id",nullable = false)` @Column 描述了数据库表中该字段的详细定义，这对于根据 JPA 注解生成数据库表结构的工具
  - name: 表示数据库表中该字段的名称，默认情形属性名称一致
  - nullable: 表示该字段是否允许为 null，默认为 true

#### User 实体类

```kotlin
@Entity
@Table(name = "t_user")
@Where(clause = "status=0")
@SQLDelete(sql="UPDATE t_user SET status = 1 WHERE auto_id = ?")
@DynamicInsert
@DynamicUpdate
class User(
        var name: String? = null,
        var age: Int? = null
) : BaseEntity()
```

User 是实际业务中用户类，继承了 BaseEntity

- `@Entity` 实体类注解
- `@Table(name = "t_user")` 实际数据库中的表名，我的表名是 t_user
- `@Where(clause = "status=0")` 我用 status 字段记录逻辑展出状态，所以要加上@Where
- @SQLDelete(sql="UPDATE t_user SET status = 1 WHERE auto_id = ?") 覆盖默认 delete 的 sql 语句，默认的 delete 是物理删除，但是国内现状是物理删除不存在的，只有逻辑删除。
- `@DynamicInsert`,`@DynamicUpdate`只插入更新已更改字段

### Repository 构建

```kotlin
interface UserRepository : JpaRepository<User, Long>{

    @Transactional
    @Modifying
    @Query("update User set age = ?1,lastModifyTime=current_timestamp where autoId = ?2")
    fun modifyAgeById(age: Int, autoId: Long): Int

    @Transactional
    @Modifying
    @Query("update User set age = :#{#user.age},lastModifyTime=current_timestamp where autoId = :#{#user.autoId}")
    fun modifyAgeByIdV2(@Param("user") user:User): Int

    @Transactional
    @Modifying
    fun deleteByAgeIn(age: List<Int>):Int

    @Query("select new map(autoId,name) from User")
    fun findNames():List<Map<String,Any>>

    @Query("select new com.demo.database.model.UserNameDto(autoId,name) from User")
    fun findNamesV2():List<UserNameDto>
}
```

创建的 Repository 只要继承 JpaRepository,JpaRepository 有很多内置方法。

## 数据保存与修改

```kotlin
// 保存
userRepository.save(User(name = "小王"))

// 批量保存
val users = listOf<User>(User(name = "小李",age = 18), User(name = "小白"))
userRepository.saveAll(users)

// 修改
val user = userRepository.findById(15).get()
user.age =19
userRepository.save(user)

// 批量修改
val users = userRepository.findAll()
users.map { it.age = 12 }
userRepository.saveAll(users)

// 不获取的数据，直接修改
userRepository.modifyAgeById(22, 1)

// 不获取的数据，直接修改v2
val user = User(age = 18)
user.autoId = 1
userRepository.modifyAgeByIdV2(user)

```

## 数据删除

```kotlin
// 根据id删除
userRepository.deleteById(1)

// 批量删除
userRepository.deleteByAgeIn(listOf(12,13))
```

**逻辑删除的重点 `@SQLDelete(sql="UPDATE t_user SET status = 1 WHERE auto_id = ?")`**

## 数据查询

查询方法网上已经有好多，请自行 google，这里主要介绍如何返回部分字段的方法

### 新建 接收数据类 UserNameDto

我们只要 autoId 和 name 字段

```kotlin
class UserNameDto(
    val autoId: Long,
    val name: String
)
```

###查询方法

```
// 获取部分字段
val usersMapList = userRepository.findNames()
val users = usersMapList.map {
    UserNameDto(it["0"] as Long, it["1"] as String)
}
// 获取部分字段V2
val users = userRepository.findNamesV2()
```

## 源码地址

github:[https://github.com/faner11/jpa-teaching](https://github.com/faner11/jpa-teaching)

### 结语

这个文档后续会继续更新，如果遇到问题请留言或者 Google，如果访问 Google 困难[酸酸乳传送门](https://justmysocks1.net/members/aff.php?aff=3254)
