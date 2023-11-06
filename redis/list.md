# list 顺序表

在 Redis 中，可以对列表两端插入（push）和弹出（pop），还可以获取指定范围的元素列表、获取指定索引下标的元素等，内部结构类似于 双端队列 `deque`。

> 在 list 中元素可以重复，
>
> 在 list 中负数下标表示从末尾元素开始，（-1 : 倒数第一个）
>
> 在开发中一般作为 栈 或者 队列 使用，队列的上位替代是  stream 

## 内部编码

列表类型的内部编码有两种：

- ziplist（压缩列表）：当元素个数较少且没有⼤元素时，Redis 会选用ziplist 来作为列表的内部编码实现来减少内存消耗。
- linkedlist（双向链表）：当列表类型无法满足 ziplist 的条件时，Redis 会使用 linkedlist 作为列表的内部实现。

> 1. 当元素个数较少且没有⼤元素时，内部编码为 ziplist
> 2. 当元素个数超过 512 时 或 某个元素的⻓度超过 64 字节时，内部编码为 linkedlist
>
> 注 ：以上可以通过修改配置来改变 （/etc/redis/redis.conf 中的 list-max-ziplist-size -2 配置项）

## 相关命令

### 插入数据

#### LPUSH

> 将⼀个或者多个元素从 **左侧** 依次放入（头插）到 list 中，
>
> 最后插入的元素会在头部

```
LPUSH key element [element ...]
```

**返回值：**插入后 list 的长度。 

#### LPUSHX

> 若 key 存在，则将⼀个或者多个元素从 **左侧** 依次放入（头插）到 list 中

```
LPUSHX key element [element ...]
```

**返回值：**插入后 list 的长度。

#### RPUSH

> 将⼀个或者多个元素从 **右侧** 依次放入（尾插）到 list 中
>
> 最后插入的元素会在尾部

```
RPUSH key element [element ...]
```

**返回值：**插入后 list 的长度。

#### RPUSHX

> 若 key 存在，则将⼀个或者多个元素从 **右侧** 依次放入（尾插）到 list 中

```
RPUSHX key element [element ...]
```

**返回值：**插入后 list 的长度。

#### LINSERT

> 在特定位置插⼊元素（从左往右第一个和pivot相等的element位置）
>
> before：在pivot之前插入element
>
> after：在pivot之后插入element

```
LINSERT key <BEFORE | AFTER> pivot element
```

**返回值：**插入后的 list 长度。

#### LSET

> 根据下标修改元素

```
LSET key index element
```

**返回值：**是否成功。

### 获取数据

#### LRANGE

> 获取从 start 到 stop 区间的所有元素，左闭右闭
>
> 下标支持负数 （lrange key 0 -1 : 获取key中所有元素）
>
> 若下标越界，则越界的部分为空

```
LRANGE key start stop
```

**返回值：**指定区间的元素。

#### LINDEX

> 获取下标为 index 的元素

```
LINDEX key index
```

**返回值：**取出的元素或者 `nil` 。

#### LLEN

> 获取 list 长度

```
LLEN key
```

**返回值：**list 的长度。



### 删除数据

#### LPOP

> 从 list 左侧取出元素（即头删）

```
LPOP key
```

**返回值：**取出的元素或者 `nil  `。

#### RPOP

> 从 list 右侧取出元素（即尾删）

```
RPOP key
```

**返回值：**取出的元素或者 `nil` 。

#### LREM

> count > 0 ：删除从左起 count 个数和 element 相等的元素
>
> count < 0 ：删除从右起 |count| 个数和 element 相等的元素
>
> count = 0 ：删除所有和 element 相等的元素

```
LREM key count element
```

**返回值：**成功删除的元素个数。

#### LTRIM

> 删除 start 和 stop 之外的元素

```
LREM key start stop
```

**返回值：**是否成功。

### 阻塞式命令

当list中为空时，pop命令将会 **阻塞等待** 一段时间，直到 **有数据到来** 或者 **超时** 。

阻塞并非完全阻塞，其他命令照样执行。

#### BLPOP

> LPOP 的阻塞版本，并且可以同时指定多个 key，只要有一个 list 满足条件就返回
>
> timeout ：阻塞的最长时间（秒）

```
BLPOP key [key ...] timeout
```

**返回值：**取出的元素（二元组：key + element）或者 `nil`。

#### BRPOP

> RPOP 的阻塞版本，并且可以同时指定多个 key，只要有一个 list 满足条件就返回
>
> timeout ：阻塞的最长时间（秒） 

```
BRPOP key [key ...] timeout
```

**返回值：**取出的元素（二元组：key + element）或者 `nil`。