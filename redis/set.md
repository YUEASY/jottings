# set 集合

集合类型也是保存多个字符串类型的元素的，但和`list`类型不同的是，集合中元素之间是无序的且不允许重复。

> 支持多个集合取交集、并集、差集
>
> set 中元素之间是无序的
>
> set 中元素不允许重复

## 内部编码

集合类型的内部编码有两种：

- `intset`（整数集合）：当集合中的元素都是整数并且元素的个数小于 set-max-intset-entries 配置（默认 512 个）时，Redis 会选用 intset 来作为集合的内部实现，从而减少内存的使用。
- `hashtable`（哈希表）：当集合类型无法满足 intset 的条件时，Redis 会使用 hashtable 作为集合的内部实现。

>1. 当元素个数较少并且都为整数时，内部编码为 intset
>2. 当元素个数超过 512 个，内部编码为 hashtable
>3. 当存在元素不是整数时，内部编码为 hashtable

## 相关命令

### 插入数据

#### SADD

> 将⼀个或者多个元素添加到 set 中
>
> 注：重复的元素无法添加

```
SADD key member [member ...]
```

**返回值：**本次添加成功的元素个数。

### 获取数据

#### SMEMBERS

> 获取⼀个 set 中的所有元素
>
> 注：元素间的顺序是无序的

```
SMEMBERS key
```

**返回值：**所有元素的列表。

#### SISMEMBER

> 判断⼀个元素在不在 set 中

```
SISMEMBER key member
```

**返回值：**1 表示元素在 `set` 中。0 表示元素不在 `set `中或者 `key` 不存在。

#### SCARD

> 获取⼀个 set 的基数（cardinality）
>
> 即 set 中的元素个数

```
SCARD key
```

**返回值：**`set` 内的元素个数。

#### SRANDMEMBER

> 随机获取 set 中一个元素（不重复）

```
SRANDMEMBER key [count]
```

**返回值：**随机元素。

### 删除数据

#### SPOP

>从 set 中**随机删除**并返回⼀个或者多个元素
>
>注：由于 set 内的元素是无序的，所以取出哪个元素实际是未定义行为，即随机的。

```
 SPOP key [count]
```

**返回值：**取出的元素。

#### SREM

> 将指定的元素从 set 中删除

```
SREM key member
```

**返回值：**本次操作删除的元素个数。

#### SMOVE

> 将⼀个元素从源 set 取出并放入目标 set 中
>
> source	-->	destination
>
> 注：
>
> - 若目标 set 中已经有该元素，则仅删去源 set 中的元素，视为成功
>
> - 若源 set 中没有该元素，则视为失败

```
SMOVE source destination member
```

**返回值：**1 表示移动成功，0 表示失败。

### 集合操作

#### 交 SINTER

> 获取给定 set 的交集中的元素

```
SINTER key [key ...]
```

**返回值：**交集的元素。

#### SINTERSTORE

> 获取给定 set 的交集中的元素并保存到 destination 即目标 set 中 

```
SINTERSTORE destination key [key ...]
```

**返回值：**交集的元素个数。

#### 并 SUNION

> 获取给定 set 的并集中的元素

```
SUNION key [key ...]
```

**返回值：**并集的元素。

#### SUNIONSTORE

> 获取给定 set 的并集中的元素并保存到 destination 即目标 set 中 

```
SUNIONSTORE destination key [key ...]
```

**返回值：**并集的元素个数。

#### 差 SDIFF

> 获取给定 set 的差集中的元素
>
> 注：顺序很重要，前者存在而后者没有的元素为差

```
SDIFF key [key ...]
```

**返回值：**差集的元素。

#### SDIFFSTORE

> 获取给定 set 的差集中的元素并保存到 destination 即目标 set 中 

```
SDIFFSTORE destination key [key ...]
```

**返回值：**差集的元素个数。