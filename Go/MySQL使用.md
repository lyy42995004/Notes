MySQL 作为目前最流行的开源关系型数据库，其 SQL 语法体系已形成行业标准，相关知识体系庞大且成熟，本文不再对 SQL 基础进行详细展开，建议尚未掌握的读者先行系统学习。本文聚焦于**如何使用 Go 语言进行 MySQL 数据库操作**，旨在为开发者提供实践层面的技术指引。

在实际项目开发中，直接使用数据库驱动进行操作的场景较少，更多开发者会选择采用 ORM（对象关系映射）框架来简化开发流程。本文选用的`sqlx`库，作为 Go 标准`sql`库的增强版本，虽然不具备完整的 ORM 功能，但凭借**简洁高效、轻量灵活**的特性，在中小型项目或对性能要求较高的场景中备受青睐。若更倾向于功能完备的 ORM 解决方案，推荐探索`Gorm`、`Xorm`、`Ent`等业界知名的 ORM 框架，它们在自动化映射、复杂查询构建等方面表现出色，能显著提升开发效率。

# 安装

首先需要安装Go的MySQL驱动：

```bash
$ go get github.com/jmoiron/sqlx
```

`sqlx`或者说标准库`database/sql`支持的数据库不止 MySQL，任何实现了`driver.Driver`接口的类型都支持，比如：

- PostgreSQL
- Oracle
- MariaDB
- SQLite
- 等其他关系数据库

要使用对应的数据库，就需要实现数据库驱动，驱动可以是你自己写的，也可以是第三方库，在使用之前你就要先使用`sql.Register`注册驱动，然后才能使用。不过一般下载的驱动库都会自动注册驱动，不需要你来手动注册。

```go
func Register(name string, driver driver.Driver)
```

由于 MySQL 比较流行，也最为简单，所以本文采用 MySQL 来讲解，其他关系数据库操作起来都是大差不大差的，下载 MySQL 驱动库

```go
$ go get github.com/go-sql-driver/mysql
```

# 连接数据库

通过`sqlx.Open`函数，就可以打开一个数据库连接，它接受两个参数，第一个是驱动名称，第二个就是数据源（一般简称 DSN）。

```go
func Open(driverName, dataSourceName string) (*DB, error)
```

驱动名称就是注册驱动时使用的名称，需要保持一致，DSN 就是数据库的连接地址，每种数据库都可能会不一样，对于 MySQL 而言就是下面这样

```go
db, err := sqlx.Open("mysql", "user:pass@tcp(localhost:3306)/dbname?charset=utf8mb4&parseTime=True")
if err != nil {
    panic(err)
}
defer db.Close()
```

# 查询

**单条查询**

```go
var db *sqlx.DB

type Person struct {
   UserId   string `db:"id"`
   Username string `db:"name"`
   Age      int    `db:"age"`
   Address  string `db:"address"`
}

func init() {
    conn, err := sqlx.Open("mysql", "root:wyh246859@tcp(127.0.0.1:3306)/test")
   if err != nil {
      fmt.Println("Open mysql failed", err)
      return
   }

   db = conn
}

func main() {
   query()
   defer db.Close()
}

func query() {
   var person Person
   //查询一个是Get，多个是Select
   err := db.Get(&person, "select * from user where id = ?", "12132")
   if err != nil {
      fmt.Println("query failed:", err)
      return
   }
   fmt.Printf("query succ:%+v", person)
}

func list() {
  var perons []Person
  err := db.Select(&perons, "select * from user")
  if err != nil {
    fmt.Println("list err", err)
    return
  }
  fmt.Printf("list succ,%+v", perons)
}
```

**多条查询**

```go
var users []User
err := db.Select(&users, "SELECT id, name FROM users WHERE age > ?", 18)
```

# 插入

**单条插入**

```go
func insert() {
   result, err := db.Exec("insert into user value (?,?,?,?)", "120230", "李四", 12, "广州市")
   if err != nil {
      fmt.Println("insert err:", err)
      return
   }
   id, err := result.LastInsertId()
   if err != nil {
      fmt.Println("insert err:", err)
      return
   }
   fmt.Println("insert succ:", id)
}
```

**命名参数插入**

```go
_, err := db.NamedExec("INSERT INTO users(name, age) VALUES(:name, :age)", 
    map[string]interface{}{"name": "李四", "age": 30})
```

# 更新

```go
func update() {
   res, err := db.Exec("update user set name = ? where id = ?", "赵六", "120230")
   if err != nil {
      fmt.Println("update err:", err)
      return
   }
   eff, err := res.RowsAffected()
   if err != nil || eff == 0 {
      fmt.Println("update err:", err)
      return
   }
   fmt.Println("Update succ")
}
```

# 删除

```go
func delete() {
   res, err := db.Exec("delete from user where id = ?", "120230")
   if err != nil {
      fmt.Println("delete err:", err)
      return
   }
   eff, err := res.RowsAffected()
   if err != nil || eff == 0 {
      fmt.Println("delete err:", err)
      return
   }
   fmt.Println("delete succ")
}
```

## 事务处理

```go
func (db *DB) Begin() (*Tx, error) 	// 开始一个事务
func (tx *Tx) Commit() error		// 提交一个事务
func (tx *Tx) Rollback() error 		// 回滚一个事务
```

当开启一个事务后，为了保险都会加一句`defer tx.Rollback()`，如果如果过程出错了，就会回滚，要是事务成功提交了，这个回滚自然是无效的。

```go
func main() {
    transation, err := db.Begin()
    if err != nil {
    fmt.Println("transation err")
    }
    defer transation.Rollback()

    insert()
    query()
    update()
    query()
    delete()
    transation.Commit()
}
```

> 参考资料：
>
> [Golang 中文学习文档 - 数据库 - MySQL](https://golang.halfiisland.com/community/database/Mysql.html)

