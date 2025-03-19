ZSet 是一个有序的字符串集合，其中每个元素都关联着一个分数（score），用于决定元素在集合中的顺序。ZSet 中的元素是唯一的，但分数可以重复。集合中的元素按照分数从小到大进行排序，当分数相同时，按照元素的字典序进行排序。

# 内部实现

## 压缩列表（ziplist）

- **结构**：ziplist 是一种紧凑的连续内存块结构，在存储 ZSet 时，元素按照分数从小到大的顺序排列，成员和分数依次交替存储在内存中，每个节点包含前一个节点的长度、当前节点的长度以及具体的数据内容。
- **优点**：在元素数量较少且元素大小较小时，内存使用效率极高，因为它避免了指针带来的额外内存开销，并且是连续存储，缓存命中率高。
- **缺点**：插入和删除操作可能会导致大量的数据移动，时间复杂度为 O (n)；查找元素也需要遍历整个列表，效率较低；同时，存在连锁更新问题，即一个节点的长度变化可能引发后续多个节点的更新，影响性能。

> **在Redis7.0中，压缩列表数据结构已经废弃，交由`listpack`数据结构来实现了。**

## Listpack

- **结构**：Listpack 同样是紧凑的连续内存结构，它改进了 ziplist 的节点布局，去除了记录前一个节点长度的字段，每个节点只包含自身长度和数据内容，使得结构更加紧凑。
- **优点**：解决了 ziplist 的连锁更新问题，在插入和删除操作上性能有所提升；并且内存使用效率进一步提高，能够更有效地存储数据。
- **缺点**：和 ziplist 类似，当元素数量增多时，查找操作仍需要遍历列表，时间复杂度为 O (n)，对于大规模数据的操作性能不如 skiplist。

## 跳表（skiplist）

- **结构**：跳表是一种基于链表的有序数据结构，每个节点包含元素的成员、分数以及多层指针，通过这些多层指针可以快速跳过一些节点，从而提高查找效率；同时，搭配一个哈希表，用于存储元素成员和分数的映射关系。
- **优点**：插入、删除和查找操作的平均时间复杂度为 O (log n)，能够高效地处理大量数据；支持范围查询，可以快速获取指定分数范围内的元素；并且实现相对简单，易于维护。
- **缺点**：相比于 ziplist 和 Listpack，内存占用较大，因为每个节点需要额外的指针空间；哈希表也需要一定的内存开销；而且在元素数量较少时，其性能优势不明显，会有额外的内存和性能损耗。

# 常用命令

## 添加

**ZADD**

```
zadd key [NX|XX] [GT|LT] [CH] [INCR] score member [score member ...]
```

- 向`zset`中添加元素和分数。

- **可选参数**：

  - `NX` | `XX`

    - **`NX`**：只在成员不存在时才添加，若成员已存在，则不进行任何操作。

    - **`XX`**：只对已存在的成员进行更新，不会添加新成员。

  - `GT` | `LT`

    - **`GT`**：仅在新分数大于当前分数时才更新成员的分数。此选项只能与 `XX` 一起使用。

    - **`LT`**：仅在新分数小于当前分数时才更新成员的分数。此选项也只能与 `XX` 一起使用。

  - **`CH`**：返回此次操作中被修改的成员数量。修改包括分数变化或成员被添加。

  - **`INCR`**：将 `ZADD` 命令变为运算操作。

## 查找

**ZRANGE**

```
zrange key start stop [withscores]
```

- 返回`[min, max]`闭区间内的元素，按升序排序，如果加上`withcores`参数，每个元素的下一行是它的`socre`。

**ZREVRANGE**

```
zrevrange key start stop [withscores]
```

- 返回`[min, max]`闭区间内的元素，按降序排序。

**ZRANGEBYSCORE**

```
zrevrange key start stop [withscores]
```

- 返回`[min, max]`闭区间内的元素，按`score`升序排序。

**ZREVRANGEBYSCORE**

```
zrevrangebyscore key min max [withscores]
```

- 返回`[min, max]`闭区间内的元素，按`score`降序排序。

**ZCARD**

```
zcard key
```

-  获取`zset`的元素个数并返回。

**ZCOUNT**

```
zcount key min max
```

- 返回`[min, max]`闭区间内的元素个数，可以通过`(min (max`来设置开区间。

**ZRANK**

```
zrank key member
```

- 获取指定元素从小到大顺序的排名。

**ZREVRANK**

```
zrevrank key member
```

-  获取指定元素从大到小顺序的排名。

## 删除

**ZPOPMAX**

```
zpopmax key [count]
```

- 删除`score`最高的`count`个元素，并返回删除的元素。
- 如果多个元素的`score`相同，那么会按照`member`的字典序进行比较，字典序高的先删除。

**BZPOPMAX**

```
bzpopmax key [key ...] timeout
```

- 阻塞删除`score`最高的`count`个元素，并返回删除的元素，如果没有元素则陷入阻塞。
- `timeout`超时时间，以秒为单位。

**ZPOPMIN**

```
zpopmin key [count]
```

- 删除`score`最小的`count`个元素，并返回删除的元素。

**BZPOPMIN**

```
bzpopmin key [key ...] timeout
```

- 阻塞删除`score`最小的`count`个元素，并返回删除的元素，如果没有元素则陷入阻塞。

**ZREM**

```
zrem key member [member ...]
```

- 删除指定元素，返回成功删除元素的个数。

**ZREMRANGEBYRANK**

```
zremrangebyrank key start stop
```

- 根据排名，删除`[min, max]`闭区间内的元素。

**ZREMRANGEBYSCORE**

```
zremrangebyscore key min max
```

- 根据分数，删除`[min, max]`闭区间内的元素。

## 修改

**ZINCRBY**

```
zincrby key increment member
```

- 给指定元素的`score`增加`increment`，`increment`可以为负值和浮点数，返回增加后的结果。

**集合操作**

相比于Set类型，ZSet类型没有支持差集运算。

**ZINRERSTORE**

```
zinterstore destination numkeys key [key ...] [weights weight [weight ...]] [aggregate <sum | min | max>]
```

- 求多个集合的交集，结果保存到指定`zset`中。

- 参数：

  - `destination `：输出结果到给`zset`中。

  - `numkeys`：指定后续输入的`key`的个数。

  - `weights`（可选）：权重，每一个`zset`都配一个`weight`，计算时`score`乘对应的`weight`。

  - `aggregate`（可选）： `score`的合并方式。
    - `sum`：求和（默认值）。
    - `min`：取最小值。
    - `max`：取最大值。

**ZUNIONSTORE**

```
zunionstore destination numkeys key [key ...] [weights weight [weight ...]] [aggregate <sum | min | max>]
```

- 求多个集合的并集，结果保存到指定`zset`中。

# 应用场景

**排行榜系统**：ZSet 常用于实现各种排行榜，如游戏中的玩家得分排行榜、网站的文章阅读量排行榜等。通过将玩家或文章的 ID 作为元素，对应的得分或阅读量作为分数，就可以方便地获取排名靠前的玩家或文章。

**时间序列数据**：可以将时间戳作为分数，相关的数据作为元素，用于存储时间序列数据。例如，记录网站的访问日志，每个日志条目可以是一个元素，其对应的时间戳作为分数，这样就可以按照时间顺序快速查询和处理日志数据。

**优先级队列**：将任务的优先级作为分数，任务的描述或 ID 作为元素，就可以使用 ZSet 来实现优先级队列。在处理任务时，可以按照分数从小到大的顺序依次取出任务进行处理，从而保证高优先级的任务先被执行。