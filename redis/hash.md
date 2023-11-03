# hash哈希表

在 Redis 中，哈希类型是指值本身又是⼀个键值对结构，

形如 key = "key1"，value = { {field1, value1 }, ..., {fieldN, valueN } }

> 哈希类型中的映射关系通常称为 field-value，用于区分 Redis 整体的键值对（key-value），
>
> 注意这里的 value 是指 field 对应的值，不是键（key）对应的值，请注意 value 在不同上下
>
> ⽂的作用。

# 内部编码

哈希的内部编码有两种：

- ziplist（压缩列表）：当哈希类型元素个数小于 hash-max-ziplist-entries 配置（默认 512 个）、同时所有值都小于 hash-max-ziplist-value 配置（默认 64 字节）时，Redis 会使用 ziplist 作为哈希的内部实现，ziplist 使用更加紧凑的结构实现多个元素的连续存储，所以在节省内存方⾯比hashtable 更加优秀。

- hashtable（哈希表）：当哈希类型无法满足 ziplist 的条件时，Redis 会使用 hashtable 作为哈希的内部实现，因为此时 ziplist 的读写效率会下降，而 hashtable 的读写时间复杂度为 O(1)。

>1. 当 field 个数比较少且没有⼤的 value 时，内部编码为 ziplist
>2. 当有 value 大于 64 字节时，内部编码会转换为 hashtable
>3. 当 field 个数超过 512 时，内部编码也会转换为 hashtable
>
>注 ：以上可以通过修改配置来改变 （/etc/redis/redis.conf）



# 相关命令

## 省流版

| 命令                                  | 执行效果                                    | 时间复杂度           |
| ------------------------------------- | ------------------------------------------- | -------------------- |
| hset key field value [field value...] | 设置值                                      | O(1)                 |
| hget key field                        | 获取值                                      | O(1)                 |
| hdel key field [field ...]            | 删除 field                                  | O(k), k 是 field个数 |
| hlen key                              | 计算 field 个数                             | O(1)                 |
| hgetall key                           | 获取所有的 field-value                      | O(k), k 是 field个数 |
| hmget key field [field ...]           | 批量获取 field-value                        | O(k), k 是 field个数 |
| hmset field value [field value ...]   | 批量获取 field-value                        | O(k), k 是 field个数 |
| hexists key field                     | 判断 field 是否存在                         | O(1)                 |
| hkeys key                             | 获取所有的 field                            | O(k), k 是 field个数 |
| hvals key                             | 获取所有的 value                            | O(k), k 是 field个数 |
| hsetnx key field value                | 设置值，但必须在 field 不存在时才能设置成功 | O(1)                 |
| hincrby key field n                   | 对应 field-value +n                         | O(1)                 |
| hincrbyfloat key field n              | 对应 field-value +n                         | O(1)                 |
| hstrlen key field                     | 计算 value 的字符串⻓度                     | O(1)                 |



## 增删查改



### HSET

> 设置hash中指定的字段（field）的值（value）

```
HSET key field value [field value ...]
```

**返回值：**添加的字段的个数。

### HSETNX

> 在字段不存在的情况下，设置 hash 中的字段和值

```
HSETNX key field value
```

**返回值：**1 表⽰设置成功，0 表⽰失败。

### HGET

> 获取 hash 中指定字段的值

```
HGET key field
```

**返回值：**字段对应的值或者 nil

### HMGET

> ⼀次获取 hash 中多个字段的值

```
HMGET key field [field ...]
```

**返回值：**字段对应的值或者 nil。

注：`HMSET` 用不上，因为 `HSET` 支持插入多条。



### HEXISTS

> 判断 hash 中是否有指定的字段

```
HEXISTS key field
```

**返回值：**1 表示存在，0 表示不存在。

### HDEL

> 删除 hash 中指定的字段，即 hash 中的 field 及其对应的 key

```
HDEL key field [field ...]
```

**返回值：**本次操作删除的字段个数。

### HKEYS

> 获取 hash 中的所有字段，即对 key 对应的 hash 中所有的 field 进行遍历

```
HKEYS key
```

**返回值：**所有字段。

### HVALS

> 获取 hash 中的所有的值，即对 key 对应的 hash 中所有的 value 进行遍历

```
HVALS key
```

**返回值：**所有的值。

### HGETALL （HKEYS + HVALS）

> 获取 hash 中的所有字段以及对应的值。

```
HGETALL key
```

**返回值：**字段和对应的值。

### HLEN

> 获取 hash 中的所有字段的个数

```
 HLEN key
```

**返回值：**字段个数。



## 计数命令

### HINCRBY

> 将 hash 中字段对应的数值添加指定的值

```
HINCRBY key field increment
```

**返回值：**该字段变化之后的值

### HINCRBYFLOAT

> HINCRBY 的浮点数版本

```
HINCRBYFLOAT key field increment
```

**返回值：**该字段变化之后的值。

注 ： 无 HINCR、HDECR