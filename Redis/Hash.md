# Hash

在Redis中，Hash命令的`k-v`存储结构相当与 JAVA 中的Map<String,Map<Object,Object>>。

## HSET

```shell
HSET key field value
```

将哈希表 **key** 中的域 **field** 的值设为 **value** 。

如果 **key** 不存在，一个新的哈希表被创建并进行`HSET`操作；如果域 **field** 已经存在于哈希表中，旧值将被覆盖。

### 可用版本

```shell
 >= 2.0.0
```

### 返回值

当`HSET`命令在哈希表中新建 **field** 域，并且为它成功设置值后，返回 **1** 。

如果哈希表中域 **field** 已经存在且`HSET`命令成功使用新值覆盖了它的旧值，返回 **0** 。

### HSETNX

```shell
HSETNX key field value
```

当且仅当域 **field** 不存在时，将哈希表 **key** 中的域 **field** 的值设置为 **value** 。

若域 **field** 已经存在，那么命令将放弃执行设置操作。

如果 **key** 不存在，一个新哈希表被创建并执行`HSETNX`命令。

### 可用版本

```shell
 >= 2.0.0
```

### 返回值

`HSETNX` 命令在设置成功时返回 **1**， 在给定域已经存在而放弃执行设置操作时返回 **0** 。

## HGET

```
 HGET hash field
```

**返回哈希表中给定域的值。**

### 可用版本

```
 >= 2.0.0
```

### 返回值

`HGET` 命令在默认情况下返回给定域的值。

如果给定域不存在于哈希表中， 又或者给定的哈希表并不存在， 那么命令返回 **nil** 。

## HEXISTS

```
 HEXISTS hash field
```

查看哈希表 **key** 中，给定域 **field** 是否存在。

### 可用版本

```
 >= 2.0.0
```

### 返回值

## HDEL

```
 HDEL key field [field …]
```

删除哈希表 **key** 中的一个或多个指定域，不存在的域将被忽略。

**在Redis2.4以下的版本里，**​`HDEL`每次只能删除单个域，如果你需要在一个原子时间内删除多个域，请将命令包含在 MULTL/EXEC块内。

### 可用版本

```
 >= 2.0.0
```

### 返回值

**被成功移除的域的数量，不包括被忽略的域。**

## HLEN

```
 HLEN key
```

### 可用版本

```
 >= 2.0.0
```

### 返回值

哈希表中域的数量。当 **key** 不存在时，返回 **0** 。

## HSTRLEN

```shell
HSTRLEN key field
```

返回哈希表 **key** 中， 与给定域 **field** 相关联的值的字符串长度（string length）。

如果给定的键或者域不存在， 那么命令返回 **0**。

### 可用版本

```shell
 >= 3.2.0
```

### 返回值

一个整数

## HINCRBY

```shell
HINCRBY key field increment
```

为哈希表 **key** 中的域 **field** 的值加上增量 **increment** ，增量也可以为负数，相当于对给定域进行减法操作。

如果 **key** 不存在，一个新的哈希表被创建并执行 `HIBCRBY` 命令。

如果域 **field** 不存在，那么在执行命令前，域的值被初始化为 **0** 。

对一个储存字符串值的域 **field** 执行`HIBCRBY`命令将造成一个错误。

本操作的值被限制在 64 位(bit)有符号数字表示之内。

### 可用版本

```shell
 >= 2.0.0
```

### 返回值

执行`HIBCRBY`命令之后，哈希表 **key** 中域 **field** 的值。

## HINCRBYFLOAT

```shell
HINCRBYFLOAT key field increment
```

为哈希表 **key** 中的域 **field** 加上浮点数增量 **increment** 。

如果哈希表中没有域 **field** ，那么`HINCRBYFLOAT`会先将域 **field** 的值设为 **0** ，然后再执行加法操作。

如果键 **key** 不存在，那么`HINCRBYFLOAT`会先创建一个哈希表，再创建域 **field** ，最后再执行加法操作。

当以下任意一个条件发生时，返回一个错误：

* 域 **field** 的值不是字符串类型(因为 redis 中的数字和浮点数都以字符串的形式保存，所以它们都属于字符串类型）
* 域 **field** 当前的值或给定的增量 **increment** 不能解释(parse)为双精度浮点数(double precision floating point number)

### 可用版本

```shell
 >= 2.6.0
```

### 返回值

执行加法操作之后 **field** 域的值。

## HMSET

```shell
HMSET key field value [field value …]
```

同时将多个 **field-value** (域-值)对设置到哈希表 **key** 中，此命令会覆盖哈希表中已存在的域。

如果 **key** 不存在，一个空哈希表被创建并执行`HMSET`操作。

### 可用版本

```shell
 >= 2.0.0
```

### 返回值

如果命令执行成功，返回 **OK**。

当 **key** 不是哈希表(hash)类型时，返回一个错误。

## HMGET

```shell
HMGET key field [field ...]
```

返回哈希表 **key** 中，一个或多个给定域的值。

如果给定的域不存在于哈希表，那么返回一个 **nil** 值。

因为不存在的 **key** 被当作一个空哈希表来处理，所以对一个不存在的 **key** 进行`HMGET`操作将返回一个只带有 **nil** 值的表。

### 可用版本

```shell
 >= 2.0.0
```

### 返回值

一个包含多个给定域的关联值的表，表值的排列顺序和给定域参数的请求顺序一样。

## HKEYS

```shell
HKEYS key
```

返回哈希表 **key** 中的所有域。

### 可用版本

```shell
 >= 2.0.0
```

### 返回值

一个包含哈希表中所有域的表。当 **key** 不存在时，返回一个空表。

## HVALS

```shell
HVALS key
```

返回哈希表 **key** 中所有域的值。

### 可用版本

```shell
 >= 2.0.0
```

### 返回值

一个包含哈希表中所有值的表。当 **key** 不存在时，返回一个空表。

## HGETALL

```shell
HGETALL key
```

返回哈希表 **key** 中，所有的域和值。

在返回值里，紧跟每个域名(field name)之后是域的值(value)，所以返回值的长度是哈希表大小的两倍。

### 可用版本

```shell
 >= 2.0.0
```

### 返回值

以列表形式返回哈希表的域和域的值。若 **key** 不存在，返回空列表。

## 应用场景

* 商品详情页
* 购车车商品信息(对数量进行计算)

‍
