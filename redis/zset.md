# zset 有序集合

有序集合类型保留了集合不能有重复成员的特点。

但与集合不同的是，有序集合中的每个元素（member）都有⼀个唯⼀的浮点类型的分数（score）来作为排序依据

> 支持多个集合取交集、并集
>
> zset 中元素之间是有序的，且元素不允许重复
>
> 若分数 score 相同，则根据 member 的字典序排序
>

## 内部编码

**zset 内部总是升序的**

有序集合类型的内部编码有两种：

- ziplist（压缩列表）：当有序集合的元素个数小于 zset-max-ziplist-entries 配置（默认 128 个），同时每个元素的值都小于 zset-max-ziplist-value 配置（默认 64 字节）时，Redis 会用 ziplist 来作为有序集合的内部实现，ziplist 可以有效减少内存的使用。
  - 在 6.2之后被 类似的 listpack 代替了

- skiplist（跳表）：当 ziplist 条件不满足时，有序集合会使用 skiplist 作为内部实现，因为此时ziplist 的操作效率会下降。

>1. 当元素个数较少并且且每个元素较小时，内部编码为 ziplist
>2. 当元素个数超过 128 个，内部编码 skiplist
>3. 当某个元素⼤于 64 字节时，内部编码 skiplist

## 相关命令

### 插入数据

#### ZADD

> 添加或者更新指定的元素 member 以及关联的分数 score 到 zset 中，分数应该符合 double 类型，+inf/-inf 作为正负极限也是合法的。
>
> 注：member 与 score 并非键值对的关系，二者只是绑定在一起
>
> ZADD 的相关选项：
>
> - XX：仅用于更新已经存在的元素，不会添加新元素。
>
> - NX：仅用于添加新元素，不会更新已经存在的元素。
> - LT：当更新后的分数更小，或者元素不存在时，更新或添加。
> - GT：当更新后的分数更大，或者元素不存在时，更新或添加。
>
> - CH：默认情况下，ZADD 返回的是本次添加的元素个数；但指定这个选项之后，就会还包含本次更新的元素的个数。
>
> - INCR：此时命令类似 ZINCRBY 的效果，将元素的分数加上指定的分数。此时只能指定⼀个元素和分数。

```
ZADD key [NX | XX] [GT | LT] [CH] [INCR] score member [score member...]
```

**返回值：**本次添加成功的元素个数，

- 若指定了 `CH`， 则再追加返回更新的元素的个数。

- 若指定了 `INCR`，则返回增加后的元素分数。

### 更改分数

#### ZINCRBY

> 为指定的元素的关联分数添加指定的分数值
>
> zadd  key xx  score member 是直接更改
>
> zadd  key xx  increment member 可以达到相同的效果

```
ZINCRBY key incr increment member
```

**返回值：**增加后元素的分数。

### 获取数据

#### ZRANGE

> 返回指定区间里的元素，分数按照升序，
>
> 带上 WITHSCORES 可以把分数也返回
>
> 注：此处的 [start ,  stop]为下标构成的区间。从 0 开始，支持负数
>
> 带上 BYSCORE 则将 [start ,  stop] 作为分数区间查找。

```
ZRANGE key start stop [BYSCORE] [WITHSCORES] 
```

**返回值：**区间内的元素列表。

#### ZRANK

> 返回指定元素的排名，从 0 开始，升序

```
ZRANK key member
```

**返回值：**升序排名。

#### ZREVRANK

> 返回指定元素的排名，从 0 开始，降序

```
ZREVRANK key member
```

**返回值：**降序排名。

#### ZSCORE

> 返回指定元素的分数

```
ZSCORE key member
```

**返回值：**对应分数。

#### ZCARD

> 获取⼀个 zset 的基数（cardinality）
>
> 即 zset 中的元素个数

```
ZSCARD key
```

**返回值：**`zset` 内的元素个数。

#### ZCOUNT

> 返回分数在 min 和 max 之间的元素个数，默认情况下，min 和 max 都是包含的
>
> 可以使用 `(min` / `(max`来排除 min 或 max

```
ZCOUNT key min max
```

**返回值：**满足条件的元素列表个数。

### 删除数据

#### ZPOPMAX

> 删除并返回分数最⾼的 count 个元素

```
ZPOPMAX key [count]
```

**返回值：**分数和元素列表。

#### ZPOPMIN

> 删除并返回分数最低的 count 个元素

```
ZPOPMIN key [count]
```

**返回值：**分数和元素列表。

#### ZREM

> 删除指定的元素

```
ZREM key member [member ...]
```

**返回值：**本次操作删除的元素个数。

#### ZREMRANGEBYRANK

> 按照排序，升序删除指定范围的元素 [start ,  stop] ，左闭右闭

```
ZREMRANGEBYRANK key start stop
```

**返回值：**本次操作删除的元素个数。

#### ZREMRANGEBYSCORE

> 按照分数 score 删除指定范围的元素 [start ,  stop]，左闭右闭

```
ZREMRANGEBYSCORE key min max
```

**返回值：**本次操作删除的元素个数。

### 阻塞式命令

#### BZPOPMAX

> ZPOPMAX 的阻塞版本
>
> timeout：超时时间

```
BZPOPMAX key [key ...] timeout
```

**返回值：**元素列表。

#### BZPOPMIN

> ZPOPMIN 的阻塞版本

```
BZPOPMIN key [key ...] timeout
```



### 集合操作

**注：**6.2版本后才支持 `zinter`、`zunion`、`zdiff`

#### 交 ZINTERSTORE

> 求出给定 zset 中元素的交集并保存进目标 zset 中，在合并过程中以元素为单位进行合并，元素对应的分数按照不同的聚合方式和权重得到新的分数。
>
> numkeys：需要填写一共有几个 key 参与运算
>
> weight：权重，在计算时 key 中的 score 会乘上对应的 weight
>
> AGGREGATE：合并方式
>
> - SUM：结果取计算权重之后的总和 （默认）
> - MIN：结果取按计算权重之后的最小值
> - MAX：结果取按计算权重之后的最大值

```
ZINTERSTORE destination numkeys key [key ...] [WEIGHTS weight [weight ...]] [AGGREGATE <SUM | MIN | MAX>]
```

**返回值：**结果集合中的元素个数。

#### 并 SUNIONSTORE

> 求出给定 zset 中元素的并集并保存进目标 zset 中，在合并过程中以元素为单位进行合并，元素对应的分数按照不同的聚合方式和权重得到新的分数。
>
> numkeys：需要填写一共有几个 key 参与运算
>
> weight：权重，在计算时 key 中的 score 会乘上对应的 weight
>
> AGGREGATE：合并方式
>
> - SUM：结果取计算权重之后的总和 （默认）
> - MIN：结果取按计算权重之后的最小值
> - MAX：结果取按计算权重之后的最大值

```
ZUNIONSTORE destination numkeys key [key ...] [WEIGHTS weight [weight ...]] [AGGREGATE <SUM | MIN | MAX>]
```

**返回值：**结果集合中的元素个数。

