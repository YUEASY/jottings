## string 字符串

**特点：对字符采用utf-8，二进制流存取，可以存入所有东西（内存限制 521MB）。**

**需要在启动向加上 --raw ，才会读取中文字符时尝试转码**

## 内部编码

string类型的内部编码有 3 种：

- `int`：8 个字节的长整型。

- `embstr`：小于等于 39 个字节的字符串（包括浮点数）。

- `raw`：大于 39 个字节的字符串。

> Redis 会根据当前值的类型和长度动态决定使用哪种内部编码实现。

## 相关命令

### 单次处理

#### SET  (SET + EXPIRE)

> 将 string 类型的 value 设置到 key 中。如果 key 之前存在，则覆盖，无论原来的数据类型是什么。之前关于此 key 的 TTL 也全部失效。

```
SET key value [expiration EX seconds|PX milliseconds] [NX|XX]
```

选项：[ SET 命令⽀持多种选项来影响它的行为 ]

- `EX` seconds⸺使用秒作为单位设置 key 的过期时间。

- `PX` *milliseconds*⸺使用毫秒作为单位设置 key 的过期时间。
- `NX` ⸺ 只在 **key 不存在时才进行设置**，即如果 key 之前已经存在，设置不执行。
- `XX` ⸺ 只在 **key 存在时才进行设置**，即如果 key 之前不存在，设置不执行。

**注意：**由于带选项的 SET 命令可以被 `SETNX` 、 `SETEX` 、 `PSETEX` 等命令代替，所以之后的版本中，Redis 可能进行合并。

**返回值：**

- 如果设置成功，返回 OK。

- 如果由于 SET 指定了 NX 或者 XX 但条件不满足，SET 不会执行，并返回 (nil)。

#### GET

> 获取 key 对应的 value。如果 key 不存在，返回 nil。如果 value 的数据类型不是 string，则报错。

```
GET key
```

**返回值：**key 对应的 value，或者 `nil` 。

#### SETNX

> key 不存在时设置

```
SETNX key value
```

#### SETEX

> key 存在时设置

```
SETEX key value
```

#### PSETEX

> key 存在时设置(毫秒级)



### 批量操作

#### MSET

> ⼀次性设置多个 key 的值。

```
 MSET key value [key value ...]
```

#### MGET

> ⼀次性获取多个 key 的值。如果对应的 key 不存在或者对应的数据类型不是 string，返回 ` nil`。

```
MGET key [key ...]
```



### 计数命令

- incr		      `++ value `
- incrby		  `value += n`
- decr             ` -- value `
- decrby         `value -= n`
- incrbyfloat  `value += n(float)`

#### INCR

> 将 key 对应的 string 表示的数字加⼀。
>
> 如果 key 不存在，则视为 key 对应的 value 是 0。如果 key 对应的 string 不是⼀个整型或者范围超过了 64 位有符号整型，则报错。

```
INCR key
```

**返回值：**integer 类型的加完后的数值。

#### INCRBY

> 将 key 对应的 string 表示的数字加上对应的值。
>
> 如果 key 不存在，则视为 key 对应的 value 是 0。如果 key 对应的 string 不是⼀个整型或者范围超过了 64 位有符号整型，则报错。

```
INCRBY key decrement
```

**返回值：**integer 类型的加完后的数值。

#### DECR

> 将 key 对应的 string 表示的数字减⼀。
>
> 如果 key 不存在，则视为 key 对应的 value 是 0。如果 key 对应的 string 不是⼀个整型或者范围超过了 64 位有符号整型，则报错。

```
DECR key
```

**返回值：**integer 类型的减完后的数值。

**DECYBY**

> 将 key 对应的 string 表示的数字减去对应的值。
>
> 如果 key 不存在，则视为 key 对应的 value 是 0。如果 key 对应的 string 不是⼀个整型或者范围超过了 64 位有符号整型，则报错。

```
DECRBY key decrement
```

**返回值：**integer 类型的减完后的数值。

#### INCRBYFLOAT  

> 将 key 对应的 string 表示的浮点数 加上/减去 对应的值。
>
> 如果对应的值是负数，则视为减去对应的值。如果key 不存在，则视为 key 对应的 value 是 0。如果 key 对应的不是 string，或者不是⼀个浮点数，则报错。允许采⽤科学计数法表示浮点数。
>
> 注：1. 无 DECYBYFLOAT   2. 此浮点数实际上是double

```
INCRBYFLOAT key increment
```

**返回值：**加/减完后的数值。

注：很多存储系统和编程语⾔内部使用CAS 机制实现计数功能，会有⼀定的 CPU 开销，但在 Redis 中完全不存在这个问题，因为 Redis 是单线程架构，任何命令到了 Redis 服务端都要顺序执行。



### 字符串操作

#### appdend 

> 如果 key 已经存在并且是⼀个 string，命令会将 value 追加到原有 string 的后边。
>
> 如果 key 不存在，则效果等同于 SET 命令。

```
appdend KEY VALUE
```

**返回值：**追加完成之后 string 的⻓度。

#### getrange

> 返回 key 对应的 string 的子串（start,end），由 start 和 end 确定（左闭右闭）。
>
> 可以使用负数表示倒数。-1 代表倒数第⼀个字符，-2 代表倒数第二个，其他的与此类似。
>
> 超过范围的偏移量会根据 string 的长度调整成正确的值。

```
 getrange key start end
```

**返回值：**string 类型的子串

注：对汉字进行切分时可能会出现问题。 

#### setrange

> 覆盖字符串的⼀部分，从指定的**偏移（offset）**开始。
>
> 如果偏移超过了字符串长度，则会自动填充 0x00 （即0000 0000）为一个字符
>
> 对不存在的 key 也可以视为空字符串进行操作

```
setrange key offset value
```

 **返回值：**覆盖字符串之后字符串的长度

注：对汉字进行覆盖时可能会出现问题。

#### strlen 

> 获取字符串长度 (以字节为单位)，当 key 存放的类型不是 string 时，报错。

```
strlen key
```

