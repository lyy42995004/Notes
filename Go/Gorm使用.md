在 Go 语言社区中，数据库交互一直是开发者们关注的重点领域，不同开发者基于自身的需求和偏好，形成了两种主要的技术选型流派。一部分开发者钟情于像`sqlx`这类简洁的库，尽管其功能并非一应俱全，但它赋予开发者对 SQL 语句的绝对控制权，便于开发者将性能优化到极致，从而满足对性能有严苛要求的场景。与之相对的另一部分开发者，则更倾向于为提升开发效率而生的 ORM（对象关系映射）框架。借助 ORM，开发者能够省去诸多繁琐的数据库操作细节，极大地加速开发进程。

在 Go 语言的 ORM 领域，`gorm`无疑占据着举足轻重的地位，作为一款历史悠久且成熟的 ORM 框架，深受广大开发者的喜爱。与`gorm`类似的，还有相对年轻的`xorm`和`ent`等框架，它们各自凭借独特的优势，在 Go 语言社区中也拥有着一批忠实的用户。

本文聚焦于`gorm`框架，主要为大家介绍其基础入门知识，希望能为读者开启探索`gorm`世界的大门。若想深入了解`gorm`的更多细节，推荐阅读官方文档，其完善的中文文档为开发者提供了详尽的学习资料。

- **官方文档**：[GORM - The fantastic ORM library for Golang, aims to be developer friendly.](https://gorm.io/zh_CN/)
- **开源仓库**：[go-gorm/gorm: The fantastic ORM library for Golang, aims to be developer friendly (github.com)](https://github.com/go-gorm/gorm)

# 特点

1. **全功能 ORM**：涵盖了丰富的数据库操作功能，为开发者提供一站式解决方案。
2. **关联关系支持**：全面支持多种关联关系，如拥有一个（Has One）、拥有多个（Has Many）、属于（Belongs To）、多对多（Many To Many）、多态（Polymorphism）以及单表继承（Single-table inheritance），满足复杂业务场景下的数据关系建模需求。
3. **钩子方法**：在 Create、Save、Update、Delete、Find 等操作中均提供了钩子方法，方便开发者在数据库操作前后进行自定义逻辑处理。
4. **预加载功能**：支持`Preload`和`Joins`的预加载方式，有效减少数据库查询次数，提升数据获取效率。
5. **事务管理**：提供完善的事务管理机制，包括事务、嵌套事务、Save Point 以及 Rollback To Saved Point 等功能，确保数据操作的原子性和一致性。
6. **多种模式支持**：支持 Context、预编译模式（Prepared Statement Mode）和 DryRun 模式，为开发者提供更多的灵活性和调试便利性。
7. **高效的数据操作**：具备批量插入、FindInBatches、Find/Create with Map 等功能，并且支持使用 SQL 表达式、Context Valuer 进行 CRUD 操作，满足不同场景下的数据操作需求。
8. **强大的 SQL 构建能力**：拥有 SQL 构建器，支持 Upsert、锁机制、Optimizer/Index/Comment Hint、命名参数以及子查询等高级 SQL 特性，让开发者能够灵活构建复杂的 SQL 语句。
9. **数据库结构管理**：支持复合主键、索引和约束的创建与管理，同时提供自动迁移功能，能够根据定义的结构体自动同步数据库表结构，减少手动维护数据库结构的工作量。
10. **自定义日志**：允许开发者自定义 Logger，方便记录和跟踪数据库操作日志，便于排查问题和进行性能分析。
11. **灵活的插件扩展**：提供灵活可扩展的插件 API，例如 Database Resolver（支持多数据库、读写分离）、Prometheus 等插件，满足不同业务场景下的扩展需求。
12. **严格的测试保障**：每个特性都经过了严格的测试，确保框架的稳定性和可靠性。
13. **开发者友好**：设计理念注重开发者体验，提供简洁易懂的 API 和丰富的文档，降低开发者的学习成本。

当然，`gorm`并非完美无缺。例如，其几乎所有方法的参数都采用空接口类型，这使得参数的传递方式较为模糊，若不查阅文档，开发者很难明确在不同场景下应传递何种参数，有时可传递结构体，有时是字符串、map 或切片。此外，在许多情况下，开发者仍需自行编写 SQL 语句来满足复杂的业务需求。

其中，`gorm`作为 Go 生态中历史悠久的 ORM 框架，凭借其全面的功能支持和良好的社区活跃度，成为众多项目的首选。本文将围绕`gorm`展开基础入门介绍，旨在帮助快速上手。

# 安装

```go
$ go get -u gorm.io/gorm
```

gorm 目前支持以下几种数据库

- MySQL ：`"gorm.io/driver/mysql"`
- PostgreSQL： `"gorm.io/driver/postgres"`
- SQLite：`"gorm.io/driver/sqlite"`
- SQL Server：`"gorm.io/driver/sqlserver"`
- TIDB：`"gorm.io/driver/mysql"`，TIDB 兼容 mysql 协议
- ClickHouse：`"gorm.io/driver/clickhouse"`

本文接下来将使用 MySQL 来进行演示，使用的什么数据库，就需要安装什么驱动，这里安装 Mysql 的 gorm 驱动。

```bash
$ go get -u gorm.io/driver/mysql  # 以MySQL为例
```

然后使用 dsn（data source name）连接到数据库，驱动库会自行将 dsn 解析为对应的配置

```go
package main

import (
  "gorm.io/driver/mysql"
  "gorm.io/gorm"
  "log/slog"
)

func main() {
  dsn := "root:123456@tcp(192.168.48.138:3306)/hello?charset=utf8mb4&parseTime=True&loc=Local"
  db, err := gorm.Open(mysql.Open(dsn))
  if err != nil {
    slog.Error("db connect error", err)
  }
  slog.Info("db connect success")
}
```

或者手动传入配置

```go
package main

import (
  "gorm.io/driver/mysql"
  "gorm.io/gorm"
  "log/slog"
)

func main() {
  db, err := gorm.Open(mysql.New(mysql.Config{}))
  if err != nil {
    slog.Error("db connect error", err)
  }
  slog.Info("db connect success")
}
```

两种方法都是等价的，看自己使用习惯。

**连接配置**

通过传入`gorm.Config`配置结构体，我们可以控制 gorm 的一些行为

```go
db, err := gorm.Open(mysql.Open(dsn), &gorm.Config{})
```

# 模型

## gorm.Model

为了方便模型定义，GORM内置了一个`gorm.Model`结构体。`gorm.Model`是一个包含了`ID`, `CreatedAt`, `UpdatedAt`, `DeletedAt`四个字段的Golang结构体。

```go
// gorm.Model 定义
type Model struct {
  ID        uint `gorm:"primary_key"`
  CreatedAt time.Time
  UpdatedAt time.Time
  DeletedAt *time.Time
}
```

你可以将它嵌入到你自己的模型中：

```go
// 将 `ID`, `CreatedAt`, `UpdatedAt`, `DeletedAt`字段注入到`User`模型中
type User struct {
    gorm.Model // 内嵌gorm.Model，包含ID, CreatedAt, UpdatedAt, DeletedAt字段    
    Name      string `gorm:"type:varchar(100);not null"`
    Email     string `gorm:"type:varchar(100);uniqueIndex"`
    Age       int    `gorm:"default:18"`
    IsActive  bool   `gorm:"default:true"`
}
```

当然你也可以完全自己定义模型：

```go
// 不使用gorm.Model，自行定义模型
type User struct {
  ID   int
  Name string
}
```

## 结构体标记（tags）

使用结构体声明模型时，标记（tags）是可选项。gorm支持以下标记:

**支持的结构体标记（Struct tags）**

| 结构体标记（Tag） |                           描述                           |
| :---------------: | :------------------------------------------------------: |
|      Column       |                         指定列名                         |
|       Type        |                      指定列数据类型                      |
|       Size        |                  指定列大小, 默认值255                   |
|    PRIMARY_KEY    |                      将列指定为主键                      |
|      UNIQUE       |                      将列指定为唯一                      |
|      DEFAULT      |                       指定列默认值                       |
|     PRECISION     |                        指定列精度                        |
|     NOT NULL      |                    将列指定为非 NULL                     |
|  AUTO_INCREMENT   |                   指定列是否为自增类型                   |
|       INDEX       | 创建具有或不带名称的索引, 如果多个索引同名则创建复合索引 |
|   UNIQUE_INDEX    |         和 `INDEX` 类似，只不过创建的是唯一索引          |
|     EMBEDDED      |                     将结构设置为嵌入                     |
|  EMBEDDED_PREFIX  |                    设置嵌入结构的前缀                    |
|         -         |                        忽略此字段                        |

**关联相关标记（tags）**

|        结构体标记（Tag）         |                描述                |
| :------------------------------: | :--------------------------------: |
|            MANY2MANY             |             指定连接表             |
|            FOREIGNKEY            |              设置外键              |
|      ASSOCIATION_FOREIGNKEY      |            设置关联外键            |
|           POLYMORPHIC            |            指定多态类型            |
|        POLYMORPHIC_VALUE         |             指定多态值             |
|       JOINTABLE_FOREIGNKEY       |          指定连接表的外键          |
| ASSOCIATION_JOINTABLE_FOREIGNKEY |        指定连接表的关联外键        |
|        SAVE_ASSOCIATIONS         |    是否自动完成 save 的相关操作    |
|      ASSOCIATION_AUTOUPDATE      |   是否自动完成 update 的相关操作   |
|      ASSOCIATION_AUTOCREATE      |   是否自动完成 create 的相关操作   |
|    ASSOCIATION_SAVE_REFERENCE    | 是否自动完成引用的 save 的相关操作 |
|             PRELOAD              |    是否自动完成预加载的相关操作    |

## 主键、表名、列名的约定

**主键（Primary Key）**

GORM 默认会使用名为ID的字段作为表的主键。

```go
type User struct {
  ID   string // 名为`ID`的字段会默认作为表的主键
  Name string
}

// 使用`AnimalID`作为主键
type Animal struct {
  AnimalID int64 `gorm:"primary_key"`
  Name     string
  Age      int64
}
```

**表名（Table Name）**

表名默认就是结构体名称的复数，例如：

```go
type User struct {} // 默认表名是 `users`

// 将 User 的表名设置为 `profiles`
func (User) TableName() string {
  return "profiles"
}

func (u User) TableName() string {
  if u.Role == "admin" {
    return "admin_users"
  } else {
    return "users"
  }
}

// 禁用默认表名的复数形式，如果置为 true，则 `User` 的默认表名是 `user`
db.SingularTable(true)
```

也可以通过`Table()`指定表名：

```go
// 使用User结构体创建名为`deleted_users`的表
db.Table("deleted_users").CreateTable(&User{})

var deleted_users []User
db.Table("deleted_users").Find(&deleted_users)
//// SELECT * FROM deleted_users;

db.Table("deleted_users").Where("name = ?", "jinzhu").Delete()
//// DELETE FROM deleted_users WHERE name = 'jinzhu';
```

GORM还支持更改默认表名称规则：

```go
gorm.DefaultTableNameHandler = func (db *gorm.DB, defaultTableName string) string  {
  return "prefix_" + defaultTableName;
}
```

**列名（Column Name）**

列名由字段名称进行下划线分割来生成

```go
type User struct {
  ID        uint      // column name is `id`
  Name      string    // column name is `name`
  Birthday  time.Time // column name is `birthday`
  CreatedAt time.Time // column name is `created_at`
}
```

可以使用结构体tag指定列名：

```go
type Animal struct {
  AnimalId    int64     `gorm:"column:beast_id"`         // set column name to `beast_id`
  Birthday    time.Time `gorm:"column:day_of_the_beast"` // set column name to `day_of_the_beast`
  Age         int64     `gorm:"column:age_of_the_beast"` // set column name to `age_of_the_beast`
}
```

## 时间戳跟踪

**CreatedAt**

如果模型有 `CreatedAt`字段，该字段的值将会是初次创建记录的时间。

```go
db.Create(&user) // `CreatedAt`将会是当前时间

// 可以使用`Update`方法来改变`CreateAt`的值
db.Model(&user).Update("CreatedAt", time.Now())
```

**UpdatedAt**

如果模型有`UpdatedAt`字段，该字段的值将会是每次更新记录的时间。

```go
db.Save(&user) // `UpdatedAt`将会是当前时间

db.Model(&user).Update("name", "jinzhu") // `UpdatedAt`将会是当前时间
```

**DeletedAt**

如果模型有`DeletedAt`字段，调用`Delete`删除该记录时，将会设置`DeletedAt`字段为当前时间，而不是直接将记录从数据库中删除。

# CRUD接口

简单的列举了，创建、查询、修改、删除的使用，详情可以看[官方文档](https://gorm.io/zh_CN/docs/)。

## 创建 (Create)

```go
// 创建单个记录
user := User{Name: "张三", Age: 20}
result := db.Create(&user) 
// 执行SQL: INSERT INTO users (name, age) VALUES ('张三', 20);

// 批量创建
users := []User{
    {Name: "李四", Age: 22},
    {Name: "王五", Age: 23},
}
db.Create(&users)
// 执行SQL: INSERT INTO users (name, age) VALUES ('李四', 22), ('王五', 23);
```

## 查询 (Read)

**单条查询**

```go
var user User
db.First(&user)
// 执行SQL: SELECT * FROM users ORDER BY id LIMIT 1;

db.Where("name = ?", "张三").First(&user)
// 执行SQL: SELECT * FROM users WHERE name = '张三' ORDER BY id LIMIT 1;

db.First(&user, 10)
// 执行SQL: SELECT * FROM users WHERE id = 10;
```

**多条查询**

```go
var users []User
db.Where("age > ?", 20).Find(&users)
// 执行SQL: SELECT * FROM users WHERE age > 20;

db.Where(map[string]interface{}{"name": "张三", "age": 20}).Find(&users)
// 执行SQL: SELECT * FROM users WHERE name = '张三' AND age = 20;
```

**高级查询**

```go
db.Select("name", "age").Find(&users)
// 执行SQL: SELECT name, age FROM users;

db.Order("age desc").Find(&users)
// 执行SQL: SELECT * FROM users ORDER BY age DESC;

db.Limit(10).Offset(5).Find(&users)
// 执行SQL: SELECT * FROM users LIMIT 10 OFFSET 5;

var count int64
db.Model(&User{}).Where("age > ?", 20).Count(&count)
// 执行SQL: SELECT COUNT(*) FROM users WHERE age > 20;
```

## 更新 (Update)

```go
db.Save(&user)
// 执行SQL: UPDATE users SET name='张三', age=20 WHERE id=1;

db.Model(&user).Update("name", "李四")
// 执行SQL: UPDATE users SET name='李四' WHERE id=1;

db.Model(&user).Updates(User{Name: "李四", Age: 21})
// 执行SQL: UPDATE users SET name='李四', age=21 WHERE id=1;

db.Model(&User{}).Where("age < ?", 20).Update("name", "未成年人")
// 执行SQL: UPDATE users SET name='未成年人' WHERE age < 20;
```

## 删除 (Delete)

```go
db.Delete(&user)
// 执行SQL: DELETE FROM users WHERE id=1;

db.Delete(&User{}, 10)
// 执行SQL: DELETE FROM users WHERE id=10;

db.Where("age < ?", 20).Delete(&User{})
// 执行SQL: DELETE FROM users WHERE age < 20;
```

# 事务

**自动事务**

```go
db.Transaction(func(tx *gorm.DB) error {
    if err := tx.Create(&user1).Error; err != nil {
        return err
    }
    // 执行SQL: INSERT INTO users (name, age) VALUES ('user1', 20);
    
    if err := tx.Create(&user2).Error; err != nil {
        return err
    }
    // 执行SQL: INSERT INTO users (name, age) VALUES ('user2', 22);
    
    return nil
})
// 如果成功执行SQL: COMMIT;
// 如果失败执行SQL: ROLLBACK;
```

**手动事务**

```go
tx := db.Begin()
// 执行SQL: BEGIN;

tx.Create(&user1)
// 执行SQL: INSERT INTO users (name, age) VALUES ('user1', 20);

tx.Create(&user2)
// 执行SQL: INSERT INTO users (name, age) VALUES ('user2', 22);

tx.Commit()
// 执行SQL: COMMIT;

// 或者出错时
tx.Rollback()
// 执行SQL: ROLLBACK;
```

**嵌套事务**

```go
db.Transaction(func(tx *gorm.DB) error {
    tx.Create(&user1)
    // 执行SQL: INSERT INTO users (name, age) VALUES ('user1', 20);
    
    tx.Transaction(func(tx2 *gorm.DB) error {
        tx2.Create(&user2)
        // 执行SQL: SAVEPOINT sp1;
        // 执行SQL: INSERT INTO users (name, age) VALUES ('user2', 22);
        return errors.New("inner error")
        // 执行SQL: ROLLBACK TO sp1;
    })
    
    return nil
    // 执行SQL: COMMIT;
})
```

**保存点** (SavePoint)

```go
tx := db.Begin()
// 执行SQL: BEGIN;

tx.Create(&user1)
// 执行SQL: INSERT INTO users (name, age) VALUES ('user1', 20);

tx.SavePoint("sp1")
// 执行SQL: SAVEPOINT sp1;

tx.Create(&user2)
// 执行SQL: INSERT INTO users (name, age) VALUES ('user2', 22);

tx.RollbackTo("sp1")
// 执行SQL: ROLLBACK TO sp1;

tx.Commit()
// 执行SQL: COMMIT;
```

> **参考资料**：
>
> [GORM 指南](https://gorm.io/zh_CN/docs/)
>
> [Golang 中文学习文档 - 第三方库 - GORM](https://golang.halfiisland.com/community/pkgs/orm/gorm.html)
>
> [李文周的博客 - GORM入门指南](https://www.liwenzhou.com/posts/Go/gorm/)
>
> [李文周的博客 - GORM CRUD指南](https://www.liwenzhou.com/posts/Go/gorm-crud/)
