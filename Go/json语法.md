在 Go 语言中，**JSON 字段**（也称为 **JSON Tag**）是附加在结构体字段上的元数据，用于控制该字段在 **JSON 编码（序列化）和解码（反序列化）** 时的行为。它的语法是：

```go
type StructName struct {
    FieldName FieldType `json:"json_field_name,option1,option2,..."`
}
```

例如：

```go
type User struct {
    Name string `json:"name"`           // 字段在 JSON 中显示为 "name"
    Age  int    `json:"age,omitempty"`  // 如果 Age 是零值，JSON 中会忽略它
}
```

# JSON 字段的作用

## 指定 JSON 字段名

默认情况下，Go 结构体的字段名会直接作为 JSON 的键名（首字母大写转为小写）。但通过 `json` Tag，可以自定义 JSON 中的键名：

```go
type Message struct {
    Sender string `json:"sender"`  // Go 字段 `Sender` → JSON 键 `"sender"`
}
```

- **输入 Go 结构体**：

```go
msg := Message{Sender: "Alice"}
```

- **输出 JSON**：

```json
{"sender": "Alice"}
```

---

## 忽略空值字段

如果字段是零值（如 `0`、`""`、`nil`），加上 `omitempty` 后，该字段不会出现在 JSON 中：

```go
type User struct {
    Name string `json:"name,omitempty"`  // 如果 Name 是 ""，JSON 中不会包含该字段
    Age  int    `json:"age,omitempty"`   // 如果 Age 是 0，JSON 中不会包含该字段
}
```

- **输入 Go 结构体**：

```go
user := User{Age: 0}
```

- **输出 JSON**：

```json
{}
```

---

## 忽略字段（`-`）

如果某个字段不需要出现在 JSON 中，可以用 `-` 忽略它：

```go
type Config struct {
    Password string `json:"-"`  // 该字段不会参与 JSON 序列化
}
```

- **输入 Go 结构体**：

```go
cfg := Config{Password: "123456"}
```

- **输出 JSON**：

```json
{}
```

# `encoding/json`包

在 go 中，`encoding/json`包下提供对应的函数来进行 json 的序列化与反序列化，主要使用的有如下函数。

```go
func Marshal(v any) ([]byte, error) //将go对象序列化为json字符串

func Unmarshal(data []byte, v any) error //将json字符串反序列化为go对象
```

首先定义结构体

```go
type Person struct {
   UserId   string
   Username string
   Age      int
   Address  string
}
```

## 序列化

```go
func main() {
  person := Person{
    UserId:   "120",
    Username: "jack",
    Age:      18,
    Address:  "usa",
  }

  bytes, err := json.Marshal(person)
  if err != nil {
    fmt.Println(err)
    return
  }
  fmt.Println(string(bytes))
}
```

**结果**

```json
{ "UserId": "120", "Username": "jack", "Age": 18, "Address": "usa" }
```

---

## 字段重命名

我们可以通过结构体标签来达到重命名的效果。

```go
type Person struct {
   UserId   string `json:"id"`
   Username string `json:"name"`
   Age      int    `json:"age"`
   Address  string `json:"address"`
}
```

此时输出

```json
{ "id": "1202", "name": "jack", "age": 19, "address": "USA" }
```

----

## 缩进

序列化时默认是没有任何缩进的，这是为了减少传输过程的空间损耗，但是这并不利于人为观察，在一些情况下我们需要将其序列化成人类能够观察的形式。为此，只需要换一个函数。

```go
func main() {
   person := Person{
      UserId:   "1202",
      Username: "jack",
      Age:      19,
      Address:  "USA",
   }
   bytes, err := json.MarshalIndent(person, "", "\t")
   if err != nil {
      fmt.Println(err)
      return
   }
   fmt.Println(string(bytes))
}
```

输出如下

```json
{
  "id": "1202",
  "name": "jack",
  "age": 19,
  "address": "USA"
}
```

---

## 反序列化

在反序列化时需要注意，如果结构体有 json 标签的话，则字段名优先以 json 标签为准，否则以结构体属性名为准。

```go
func main() {
  person := Person{}

  jsonStr := "{\"id\":\"120\",\"name\":\"jack\",\"age\":18,\"address\":\"usa\"}\n"

  err := json.Unmarshal([]byte(jsonStr), &person)
  if err != nil {
    fmt.Println(err)
    return
  }
  fmt.Printf("%+v", person)
}
```

输出

```json
{UserId:120 Username:jack Age:18 Address:usa}
```

> 参考资料：
>
> [Golang 中文学习文档 标准库 encoding/json包](https://golang.halfiisland.com/essential/std/encode.html#json)

