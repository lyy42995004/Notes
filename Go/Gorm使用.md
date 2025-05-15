在 Go 语言社区中，数据库交互一直是开发者们关注的重点领域，不同开发者基于自身的需求和偏好，形成了两种主要的技术选型流派。一部分开发者钟情于像`sqlx`这类简洁的库，尽管其功能并非一应俱全，但它赋予开发者对 SQL 语句的绝对控制权，便于开发者将性能优化到极致，从而满足对性能有严苛要求的场景。与之相对的另一部分开发者，则更倾向于为提升开发效率而生的 ORM（对象关系映射）框架。借助 ORM，开发者能够省去诸多繁琐的数据库操作细节，极大地加速开发进程。

在 Go 语言的 ORM 领域，`gorm`无疑占据着举足轻重的地位，作为一款历史悠久且成熟的 ORM 框架，深受广大开发者的喜爱。与`gorm`类似的，还有相对年轻的`xorm`和`ent`等框架，它们各自凭借独特的优势，在 Go 语言社区中也拥有着一批忠实的用户。

本文聚焦于`gorm`框架，主要为大家介绍其基础入门知识，希望能为读者开启探索`gorm`世界的大门。若想深入了解`gorm`的更多细节，推荐阅读官方文档，其完善的中文文档为开发者提供了详尽的学习资料。

- **官方文档**：[GORM - The fantastic ORM library for Golang, aims to be developer friendly.](https://gorm.io/zh_CN/)
- **开源仓库**：[go-gorm/gorm: The fantastic ORM library for Golang, aims to be developer friendly (github.com)](https://github.com/go-gorm/gorm)

## 特点

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

在 gorm 中，模型与数据库表相对应，它通常由结构体的方式展现，例如下面的结构体。

```go
type User struct {
    gorm.Model        // 内嵌gorm.Model，包含ID, CreatedAt, UpdatedAt, DeletedAt字段
    Name      string `gorm:"type:varchar(100);not null"`
    Email     string `gorm:"type:varchar(100);uniqueIndex"`
    Age       int    `gorm:"default:18"`
    IsActive  bool   `gorm:"default:true"`
}
```

**字段标签**

| 标签名                 | 说明                                                         |
| ---------------------- | ------------------------------------------------------------ |
| column                 | 指定 db 列名                                                 |
| type                   | 列数据类型，推荐使用兼容性好的通用类型，例如：所有数据库都支持 bool、int、uint、float、string、time、bytes 并且可以和其他标签一起使用，例如：not null、size, autoIncrement… |
| serializer             | 指定将数据序列化或反序列化到数据库中的序列化器，例如: serializer:json/gob/unixtime |
| size                   | 定义列数据类型的大小或长度，例如 size: 256                   |
| primaryKey             | 将列定义为主键                                               |
| unique                 | 将列定义为唯一键                                             |
| default                | 定义列的默认值                                               |
| precision              | 指定列的精度                                                 |
| scale                  | 指定列大小                                                   |
| not null               | 指定列为 NOT NULL                                            |
| autoIncrement          | 指定列为自动增长                                             |
| autoIncrementIncrement | 自动步长，控制连续记录之间的间隔                             |
| embedded               | 嵌套字段                                                     |
| embeddedPrefix         | 嵌入字段的列名前缀                                           |
| autoCreateTime         | 创建时追踪当前时间，对于 int 字段，它会追踪时间戳秒数，您可以使用 nano/milli 来追踪纳秒、毫秒时间戳，例如：autoCreateTime:nano |
| autoUpdateTime         | 创建 / 更新时追踪当前时间，对于 int 字段，它会追踪时间戳秒数，您可以使用 nano/milli 来追踪纳秒、毫秒时间戳，例如：autoUpdateTime:milli |
| index                  | 根据参数创建索引，多个字段使用相同的名称则创建复合索引，查看 索引 open in new window 获取详情 |
| uniqueIndex            | 与 index 相同，但创建的是唯一索引                            |
| check                  | 创建检查约束，例如 check:age > 13，查看 约束 open in new window 获取详情 |
| <-                     | 设置字段写入的权限， <-:create 只创建、<-:update 只更新、<-:false 无写入权限、<- 创建和更新权限 |
| ->                     | 设置字段读的权限，->:false 无读权限                          |
| -                      | 忽略该字段，- 表示无读写，-:migration 表示无迁移权限，-:all 表示无读写迁移权限 |
| comment                | 迁移时为字段添加注释                                         |
| foreignKey             | 指定当前模型的列作为连接表的外键                             |
| references             | 指定引用表的列名，其将被映射为连接表外键                     |
| polymorphic            | 指定多态类型，比如模型名                                     |
| polymorphicValue       | 指定多态值、默认表名                                         |
| many2many              | 指定连接表表名                                               |
| joinForeignKey         | 指定连接表的外键列名，其将被映射到当前表                     |
| joinReferences         | 指定连接表的外键列名，其将被映射到引用表                     |
| constraint             | 关系约束，例如：OnUpdate、OnDelete                           |

# 创建

## Create

在创建新的记录时，大多数情况都会用到`Create`方法

```go
func (db *DB) Create(value interface{}) (tx *DB)
```

现有如下的结构体

```go
type Person struct {
  Id   uint `gorm:"primaryKey;"`
  Name string
}
```

创建一条记录

```go
user := Person{
    Name: "jack",
}

// 必须传入引用
db = db.Create(&user)

// 执行过程中发生的错误
err = db.Error
// 创建的数目
affected := db.RowsAffected
```

创建完成后，gorm 会将主键写入 user 结构体中，所以这也是为什么必须得传入指针。如果传入的是一个切片，就会批量创建

```go
user := []Person{
    {Name: "jack"},
    {Name: "mike"},
    {Name: "lili"},
}

db = db.Create(&user)
```

同样的，gorm 也会将主键写入切片中。当数据量过大时，也可以使用`CreateInBatches`方法分批次创建，因为生成的`INSERT INTO table VALUES (),()`这样的 SQL 语句会变的很长，每个数据库对 SQL 长度是有限制的，所以必要的时候可以选择分批次创建。

```go
db = db.CreateInBatches(&user, 50)
```

除此之外，`Save`方法也可以创建记录，它的作用是当主键匹配时就更新记录，否则就插入。

```go
func (db *DB) Save(value interface{}) (tx *DB)
```

```go
user := []Person{
    {Name: "jack"},
    {Name: "mike"},
    {Name: "lili"},
}

db = db.Save(&user)
```

## Upsert

`Save`方法只能是匹配主键，我们可以通过构建`Clause`来完成更加自定义的 upsert。比如下面这行代码

```
db.Clauses(clause.OnConflict{
    Columns:   []clause.Column{{Name: "name"}},
    DoNothing: false,
    DoUpdates: clause.AssignmentColumns([]string{"address"}),
    UpdateAll: false,
}).Create(&p)
```

它的作用是当字段`name`冲突后，更新字段`address`的值，不冲突的话就会创建一个新的记录。也可以在冲突的时候什么都不做

```
db.Clauses(clause.OnConflict{
    Columns:   []clause.Column{{Name: "name"}},
    DoNothing: true,
}).Create(&p)
```

或者直接更新所有字段



```
db.Clauses(clause.OnConflict{
    Columns:   []clause.Column{{Name: "name"}},
    UpdateAll: true,
}).Create(&p)
```

在使用 upsert 之前，记得给冲突字段添加索引。

> **参考资料**：
>
> [Golang 中文学习文档 - 第三方库 - GORM](https://golang.halfiisland.com/community/pkgs/orm/gorm.html)
