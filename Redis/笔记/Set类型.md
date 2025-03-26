Set 类型由多个不重复的字符串元素构成，元素之间没有特定的顺序。在 Set 中，每个元素只会出现一次，插入重复元素时不会产生额外的存储。

# 内部实现

## 整数集合（intset）

当集合中的元素都是整数，并且元素数量较少时，Redis 会使用整数集合来存储。整数集合是一种紧凑的数组结构，能够高效地存储整数元素，节省内存空间。

## 哈希表（hashtable）

当集合中的元素包含非整数或者元素数量较多时，Redis 会采用哈希表来实现 Set。哈希表通过哈希函数将元素映射到不同的桶中，能快速地进行插入、查找和删除操作，平均时间复杂度为 O (1)。

# 常用命令

## 增加

**SADD**

```
sadd key member [member ...]
```

- 将一个或多个元素添加到`set`中，如果`key`不存在则创建一个集合，返回本次操作添加成功的元素个数。

**SMOVE**

```
smove source destination member
```

- 将一个元素从源`set`移动到目的`set`中，返回`0`表示移动失败，返回`1`表示成功。
- **参数**：
  - `source`：代表源集合的键名。
  - `destination`：表示目标集合的键名。
  - `member`：是需要从源集合移动到目标集合的元素。

## 查找

**SMEMBERS**

```
smembers key
```

- 以无序形式获取`set`中的所有元素。

**SCARD**

```
scard key
```

- 获取集合的元素个数。

**SRANDMEMBER**

```
srandmember key [count]
```

- 从集合`key`中随机选出`count`个元素。

## 删除

**SPOP**

```
spop key [count]
```

- 从集合`key`中随机选出`count`个元素删除。

**SREM**

```
srem key member [member ...]
```

- 删除`set`中的指定元素，回删除成功的元素个数。

## 集合运算

**交集运算**

**SINTER**

```
sinter key [key ...]
```

- 求出多个集合的交集，返回交集的结果。

**SINTERSTORE**

```
sinterstore destination key [key ...]
```

- 求出多个集合的交集，并输出到新集合`destination`中。

**并集运算**

**SUNION**

```
sunion key [key ...]
```

- 求出多个集合的并集，返回并集的结果。

**SUNIONSTORE**

```
sunionstore destination key [key ...]
```

- 求出多个集合的并集，并输出到新集合`destination`中。

**差集运算**

**SDIFF**

```
sdiff key [key ...]
```

- 求出多个集合的差集，返回差集的结果。

**SDIFFSTORE**

```
sdiffstore key [key ...]
```

- 求出多个集合的差集，并输出到新集合`destination`中。

# 应用场景

**去重功能**：利用 Set 元素的唯一性，可用于对数据进行去重。例如，在统计网站的独立访客数量时，将每个访客的 ID 添加到 Set 中，无论同一访客访问多少次，都只会记录一次。

**标签系统**：在社交网络或内容管理系统中，可使用 Set 来存储标签。每个内容可以关联一个 Set，Set 中的元素就是该内容的标签。通过集合运算，可以方便地找出具有相同标签的内容。

**抽奖系统**：可以将参与抽奖的用户 ID 存储在 Set 中，然后使用`SRANDMEMBER`或`SPOP`命令随机选取中奖用户。`SRANDMEMBER`会随机返回集合中的一个或多个元素，但不会移除元素；`SPOP`会随机移除并返回集合中的一个元素。