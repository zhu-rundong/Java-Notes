# SortedSet

**SortedSet** 是一种类似于集合和哈希之间的混合的数据类型。像 [Set](./Set.md) 一样，有序集合由唯一的，不重复字符串元素组成，所以在某种意义上也是一个排序的集合。

**SortedSet** 中每个元素与浮点值相关联，称为分数（这也是该类型也类似于散列的原因，因为每个元素都被映射到值）。Redis 通过分数来为集合中的成员进行从小到大（从左到右）的排序。

**存储结构：**[SortSet存储结构](./SortSet存储结构.md)

## ZADD

```shell
ZADD key score member [[score member] [score member] ...]
```

将一个或多个 **member** 元素及其 **score** 值加入到有序集 **key** 当中。

如果某个 **member** 已经是有序集的成员，那么更新这个 **member** 的 **score** 值，并通过重新插入这个 **member** 元素，来保证该 **member** 在正确的位置上。

**score** 值可以是整数值或双精度浮点数。

如果 **key** 不存在，则创建一个空的有序集并执行`ZADD`操作。

当 **key** 存在但不是有序集类型时，返回一个错误。

### 可用版本

```shell
>= 1.2.0
```

### 返回值

被成功添加的新成员的数量，不包括那些被更新的、已经存在的成员。

## 示例

```shell
127.0.0.1:6380> zadd k1 8 u8 3 u3 5 u5
(integer) 3
```

## ZSCORE

```shell
ZSCORE key member
```

返回有序集 **key** 中，成员 **member** 的 **score** 值。

如果 **member** 元素不是有序集 **key** 的成员，或 **key** 不存在，返回 **nil**

### 可用版本

```shell
>= 1.2.0
```

### 返回值

被成功添加的新成员的数量，不包括那些被更新的、已经存在的成员。

## 示例

```shell
127.0.0.1:6380> zscore k1 u3
"3"
```

## ZINCRBY

```shell
ZINCRBY key increment member
```

为有序集 **key** 的成员 **member** 的 **score** 值加上增量 **increment** 。

可以通过传递一个负数值 **increment** ，让 **score** 减去相应的值，比如 **ZINCRBY** **key** **-5** **member** ，就是让 **member** 的 **score** 值减去 **5** 。

当 **key** 不存在，或 **member** 不是 **key** 的成员时， **ZINCRBY** **key** **increment** **member** 等同于 **ZADD** **key** **increment** **member** 。

当 **key** 不是有序集类型时，返回一个错误。

**score** 值可以是整数值或双精度浮点数。

### 可用版本

```shell
>= 1.2.0
```

### 返回值

**member** 成员的新 **score** 值，以字符串形式表示。

## 示例

```shell
127.0.0.1:6380> zincrby k1 0.25 u5
"5.25"
```

## ZCARD

```shell
ZCARD key
```

返回有序集 **key** 的基数。

### 可用版本

```shell
>= 1.2.0
```

### 返回值

当 **key** 存在且是有序集类型时，返回有序集的基数；当 **key** 不存在时，返回 **0** 。

## 示例

```shell
127.0.0.1:6380> zcard k1
(integer) 3
```

## ZCOUNT

```shell
ZCOUNT key min max
```

返回有序集 **key** 中， **score** 值在 **min** 和 **max** 之间(默认包括 **score** 值等于 **min** 或 **max** )的成员的数量。

关于参数 **min** 和 **max** 的详细使用方法，参考 [区间及无限](#rangeAndInfiniteId)。

### 可用版本

```shell
>= 2.0.0
```

### 返回值

**score** 值在 **min** 和 **max** 之间的成员的数量。

## 示例

```shell
127.0.0.1:6380> zrange k1 0 -1 withscores
1) "u3"
2) "3"
3) "u5"
4) "5.25"
5) "u8"
6) "8"
127.0.0.1:6380> zcount k1 3 6   # 分值在 3 -6 之间的元素数量（闭区间）
(integer) 2
127.0.0.1:6380> zcount k1 (3 6  # 分值在 3 -6 之间的元素数量（前开后闭区间）
(integer) 1
127.0.0.1:6380> zcount k1 -inf +inf  # 整个集合的元素数量
(integer) 3
```

## ZRANGE

```shell
ZRANGE key start stop [WITHSCORES]
```

返回有序集 **key** 中，指定区间内的成员。

其中成员的位置按 **score** 值递增(从小到大)来排序。

具有相同 **score** 值的成员按[字典序](#dictSorderId)来排列。

如果你需要成员按 **score** 值递减(从大到小)来排列，请使用 `ZREVRANGE` 命令。

下标参数说明：[下标说明](#subscriptId)。

可以通过使用 **WITHSCORES** 选项，来让成员和它的 **score** 值一并返回，返回列表以 **value1,score1,** **...,** **valueN,scoreN** 的格式表示。

客户端库可能会返回一些更复杂的数据类型，比如数组、元组等。

### 可用版本

```shell
>= 1.2.0
```

### 返回值

指定区间内，带有 **score** 值(可选)的有序集成员的列表。

## 示例

```shell
127.0.0.1:6380> zrange k1 0 -1       # 显示整个有序集合的成员
1) "u3"
2) "u5"
3) "u8"
127.0.0.1:6380> zrange k1 0 -1 withscores    # 显示整个有序集合的成员，包含分数
1) "u3"
2) "3"
3) "u5"
4) "5.25"
5) "u8"
6) "8"
127.0.0.1:6380> zrange k1 1 2        # 显示有序集下标区间 1 至 2 的成员
1) "u5"
2) "u8"

```

## ZREVRANGE

```shell
ZREVRANGE key start stop [WITHSCORES]
```

返回有序集 **key** 中，指定区间内的成员。其中成员的位置按 **score** 值递减(从大到小)来排列。

具有相同 **score** 值的成员[字典序](#dictSorderId)排列。

除了成员按 **score** 值递减的次序排列这一点外，`ZREVRANGE`命令的其他方面和 `ZRANGE` 命令一样。

### 可用版本

```shell
>= 1.2.0
```

### 返回值

指定区间内，带有 **score** 值(可选)的有序集成员的列表。

## 示例

```shell
127.0.0.1:6380> zrange k1 0 -1 withscores           # 递增排列
1) "u3"
2) "3"
3) "u5"
4) "5.25"
5) "u8"
6) "8"
127.0.0.1:6380> zrevrange k1 0 -1 withscores        # 递减排列
1) "u8"
2) "8"
3) "u5"
4) "5.25"
5) "u3"
6) "3"
```

## ZRANGEBYSCORE

```shell
ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT offset count]
```

返回有序集 **key** 中，所有 **score** 值介于 **min** 和 **max** 之间（[区间及无限](#rangeAndInfiniteId)）的成员。有序集成员按 **score** 值递增(从小到大)次序排列。

具有相同 **score** 值的成员按[字典序](#dictSorderId)来排列(该属性是有序集提供的，不需要额外的计算)。

可选的 **LIMIT** 参数指定返回结果的数量及区间(就像SQL中的 **SELECT** **LIMIT** **offset,** **count** )，注意当 **offset** 很大时，定位 **offset** 的操作可能需要遍历整个有序集，此过程最坏复杂度为 O(N) 时间。

可选的 **WITHSCORES** 参数决定结果集是单单返回有序集的成员，还是将有序集成员及其 **score** 值一起返回。

### 可用版本

```shell
>= 1.0.5
```

### 返回值

指定区间内，带有 **score** 值(可选)的有序集成员的列表。

## 示例

```shell
127.0.0.1:6380> zrangebyscore k1 -inf +inf           # 显示整个有序集
1) "u3"
2) "u5"
3) "u8"
127.0.0.1:6380> zrangebyscore k1 -inf 5 withscores  # 显示 score<=5 的元素
1) "u3"
2) "3"
127.0.0.1:6380> zrangebyscore k1 (5 +inf withscores # 显示 score>5 的元素
1) "u5"
2) "5.25"
3) "u8"
4) "8"
```

## ZREVRANGEBYSCORE

```shell
ZREVRANGEBYSCORE key max min [WITHSCORES] [LIMIT offset count]
```

返回有序集 **key** 中， **score** 值介于 **max** 和 **min** 之间（[区间及无限](#rangeAndInfiniteId)）的所有的成员。有序集成员按 **score** 值递减(从大到小)的次序排列。

具有相同 **score** 值的成员按[字典序](#dictSorderId)排列。

除了成员按 **score** 值递减的次序排列这一点外， `ZREVRANGEBYSCORE`命令的其他方面和 `ZRANGEBYSCORE` 命令一样。

### 可用版本

```shell
>= 2.2.0
```

### 返回值

指定区间内，带有 **score** 值(可选)的有序集成员的列表。

## 示例

```shell
127.0.0.1:6380> zrevrangebyscore k1 +inf -inf           # 显示整个有序集及元素的 score 值
1) "u8"
2) "u5"
3) "u3"
127.0.0.1:6380> zrevrangebyscore k1 +inf 5 withscores   # 显示 score>=5 的元素
1) "u8"
2) "8"
3) "u5"
4) "5.25"
127.0.0.1:6380> zrevrangebyscore k1 (5 -inf withscores   # 显示 score<5 的元素
1) "u3"
2) "3"
```

## ZRANK

```shell
ZRANK key member
```

返回有序集 **key** 中成员 **member** 的排名。其中有序集成员按 **score** 值递增(从小到大)顺序排列。

排名以 **0** 为底，也就是说， **score** 值最小的成员排名为 **0** 。

### 可用版本

```shell
>= 2.0.0
```

### 返回值

如果 **member** 是有序集 **key** 的成员，返回 **member** 的排名；如果 **member** 不是有序集 **key** 的成员，返回 **nil** 。

## 示例

```shell
127.0.0.1:6380> zrange k1 0 -1 withscores
1) "u3"
2) "3"
3) "u5"
4) "5.25"
5) "u8"
6) "8"
127.0.0.1:6380> zrank k1 u3
(integer) 0
127.0.0.1:6380> zrank k1 u8
(integer) 2
```

## ZREVRANK

```shell
ZREVRANK key member
```

返回有序集 **key** 中成员 **member** 的排名。其中有序集成员按 **score** 值递减(从大到小)排序。

排名以 **0** 为底，也就是说， **score** 值最大的成员排名为 **0** 。

### 可用版本

```shell
>= 2.0.0
```

### 返回值

如果 **member** 是有序集 **key** 的成员，返回 **member** 的排名；如果 **member** 不是有序集 **key** 的成员，返回 **nil** 。

## 示例

```shell
127.0.0.1:6380> zrevrange k1 0 -1 withscores
1) "u8"
2) "8"
3) "u5"
4) "5.25"
5) "u3"
6) "3"
127.0.0.1:6380> zrevrank k1 u3
(integer) 2
127.0.0.1:6380> zrevrank k1 u8
(integer) 0
```

## ZREM

```shell
ZREM key member [member ...]
```

移除有序集 **key** 中的一个或多个成员，不存在的成员将被忽略。

当 **key** 存在但不是有序集类型时，返回一个错误。

### 可用版本

```shell
>= 1.2.0
```

### 返回值

被成功移除的成员的数量，不包括被忽略的成员。

## ZREMRANGEBYRANK

```shell
ZREMRANGEBYRANK key start stop
```

移除有序集 **key** 中，指定排名(rank)区间内的所有成员。

区间分别以下标参数 **start** 和 **stop** 指出，包含 **start** 和 **stop** 在内（[下标说明](#subscriptId)）。

### 可用版本

```shell
>= 2.0.0
```

### 返回值

被移除成员的数量。

## ZREMRANGEBYSCORE

```shell
ZREMRANGEBYSCORE key min max
```

移除有序集 **key** 中，所有 **score** 值介于 **min** 和 **max** 之间（[区间及无限](#rangeAndInfiniteId)）的成员。

### 可用版本

```shell
>= 1..0
```

### 返回值

被移除成员的数量。

## ZRANGEBYLEX

```shell
ZRANGEBYLEX key min max [LIMIT offset count]
```

当有序集合的所有成员都具有相同的分值时， 有序集合的元素会根据成员的字典序（lexicographical ordering）来进行排序， 而这个命令

则可以返回给定的有序集合键 **key** 中， 值介于**min** 和 **max** 之间（[区间范围](#rangeId)）的成员。

如果有序集合里面的成员带有不同的分值， 那么命令返回的结果是未指定的（unspecified）。

可选的 **LIMIT** 参数用于获取指定范围内的匹配元素(就像SQL中的 **SELECT** **LIMIT** **offset,** **count** )，注意当 **offset** 很大时，定位 **offset** 的操作可能需要遍历整个有序集，此过程最坏复杂度为 O(N) 时间。

### 可用版本

```shell
>= 2.8.9
```

### 返回值

一个列表，列表里面包含了有序集合在指定范围内的成员。

### 示例

```shell
127.0.0.1:6380> ZADD myzset 0 a 0 b 0 c 0 d 0 e 0 f 0 g
(integer) 7
127.0.0.1:6380> ZRANGEBYLEX myzset - [c         # 返回 元素字典序<=c 的元素
1) "a"
2) "b"
3) "c"
127.0.0.1:6380> ZRANGEBYLEX myzset [aaa (g     # 返回 aaa<=元素字典序<g 的元素
1) "b"
2) "c"
3) "d"
4) "e"
5) "f"
```

## ZLEXCOUNT

```shell
ZLEXCOUNT key min max
```

对于一个所有成员的分值都相同的有序集合键 **key** 来说， 这个命令会返回该集合中，值介于**min** 和 **max** （[区间范围](#rangeId)）范围内的元素数量。

### 可用版本

```shell
>= 2.8.9
```

### 返回值

整数回复：指定范围内的元素数量。

### 示例

```shell
127.0.0.1:6380> ZADD myzset 0 a 0 b 0 c 0 d 0 e 0 f 0 g
(integer) 7
127.0.0.1:6380> ZRANGEBYLEX myzset - [c         # 返回 元素字典序<=c 的元素
1) "a"
2) "b"
3) "c"
127.0.0.1:6380> ZRANGEBYLEX myzset [aaa (g     # 返回 aaa<=元素字典序<g 的元素
1) "b"
2) "c"
3) "d"
4) "e"
5) "f"
127.0.0.1:6380> ZLEXCOUNT myzset - [c        # 返回 元素字典序<=c 的元素数量
(integer) 3
127.0.0.1:6380> ZLEXCOUNT myzset [aaa (g     # 返回 aaa<=元素字典序<g 的元素数量
(integer) 5
```

## ZREMRANGEBYLEX

```shell
ZREMRANGEBYLEX key min max
```

对于一个所有成员的分值都相同的有序集合键 **key** 来说， 这命令移除该集合中， 成员介于 **min** 和 **max** （[区间范围](#rangeId)）范围内的所有元素。

### 可用版本

```shell
>= 2.8.9
```

### 返回值

整数回复：被移除的元素数量。

### 示例

```shell
127.0.0.1:6380> ZREMRANGEBYLEX myzset - [c  # 移除 元素字典序<=c 的元素
(integer) 3
127.0.0.1:6380> ZRANGEBYLEX myzset - +
1) "d"
2) "e"
3) "f"
4) "g"
```

## ZUNIONSTORE

```shell
ZUNIONSTORE destination numkeys key [key ...] [WEIGHTS weight [weight ...]] [AGGREGATE SUM|MIN|MAX]
```

计算给定的一个或多个有序集的并集，其中给定 **key** 的数量必须以 **numkeys** 参数指定，并将该并集(结果集)储存到 **destination** 。

默认情况下，结果集中某个成员的 **score** 值是所有给定集下该成员 **score** 值之 **和**。

**WEIGHTS**

使用 **WEIGHTS** 选项，你可以为 **每个** 给定有序集 **分别** 指定一个乘法因子(multiplication factor)，每个给定有序集的所有成员的 **score** 值在传递给聚合函数(aggregation function)之前都要先乘以该有序集的因子。

如果没有指定 **WEIGHTS** 选项，乘法因子默认设置为 **1** 。

**AGGREGATE**

使用 **AGGREGATE** 选项，你可以指定并集的结果集的聚合方式。

默认使用的参数 **SUM** ，可以将所有集合中某个成员的 **score** 值之 **和** 作为结果集中该成员的 **score** 值；使用参数 **MIN** ，可以将所有集合中某个成员的 **最小 score** 值作为结果集中该成员的 **score** 值；而参数 **MAX** 则是将所有集合中某个成员的 **最大** **score** 值作为结果集中该成员的 **score** 值。

### 可用版本

```shell
>= 2.0.0
```

### 返回值

保存到 **destination** 的结果集的基数。

### 示例

```shell
127.0.0.1:6380> zadd k1 70 u1 90 u2 60 u3
(integer) 3
127.0.0.1:6380> zadd k2 90 u1 80 u2 100 u4
(integer) 3
127.0.0.1:6380> ZUNIONSTORE unkey1 2 k1 k2
(integer) 4
127.0.0.1:6380> ZRANGE unkey1 0 -1 withscores   # u3、u4为新增元素，默认放在最前面
1) "u3"
2) "60"
3) "u4"
4) "100"
5) "u1"
6) "160"
7) "u2"
8) "170"
127.0.0.1:6380> ZUNIONSTORE unkey1 2 k1 k2 weights 1 0.5    # k1、k2权重分别为 1 和 0.5
(integer) 4
127.0.0.1:6380> ZRANGE unkey1 0 -1 withscores
1) "u4"
2) "50"
3) "u3"
4) "60"
5) "u1"
6) "115"
7) "u2"
8) "130"
127.0.0.1:6380> ZUNIONSTORE unkey1 2 k1 k2 aggregate max  # 取最大值
(integer) 4
127.0.0.1:6380> ZRANGE unkey1 0 -1 withscores
1) "u3"
2) "60"
3) "u1"
4) "90"
5) "u2"
6) "90"
7) "u4"
8) "100"
```

## ZINTERSTORE

```shell
ZINTERSTORE destination numkeys key [key ...] [WEIGHTS weight [weight ...]] [AGGREGATE SUM|MIN|MAX]
```

计算给定的一个或多个有序集的交集，其中给定 **key** 的数量必须以 **numkeys** 参数指定，并将该交集(结果集)储存到 **destination** 。

默认情况下，结果集中某个成员的 **score** 值是所有给定集下该成员 **score** 值之和.

关于 **WEIGHTS** 和 **AGGREGATE** 选项的描述，参见 `ZUNIONSTORE` 命令。

### 可用版本

```shell
>= 2.0.0
```

### 返回值

保存到 **destination** 的结果集的基数。

### 示例

```shell
127.0.0.1:6380> zadd k1 70 u1 90 u2 60 u3 50 u4
(integer) 4
127.0.0.1:6380> zadd k2 60 u1 100 u2 80 u4
(integer) 3
127.0.0.1:6380> ZINTERSTORE inkey1 2 k1 k2
(integer) 3
127.0.0.1:6380> ZRANGE inkey1 0 -1 withscores
1) "u1"
2) "130"
3) "u4"
4) "130"
5) "u2"
6) "190"
127.0.0.1:6380> ZINTERSTORE inkey1 2 k1 k2 weights 1 0.5 aggregate min
(integer) 3
127.0.0.1:6380> ZRANGE inkey1 0 -1 withscores
1) "u1"
2) "30"
3) "u4"
4) "40"
5) "u2"
6) "50"
```

## 应用场景

* 微博、抖音热搜

  ```shell
  # 点击一次，加一
  ZINCRBY movie:20220219 1 movie1
  ZINCRBY movie:20220219 1 movie1
  ZINCRBY movie:20220219 1 movie2
  # 前十，按播放量倒序排序
  ZREVRANGE movie:20220219 0 9 withscores
  ```
* 根据商品销售对商品进行排序显示

## <a id="rangeAndInfiniteId">区间及无限</a>

**min** 和 **max** 可以是 **-inf** 和 **+inf** ，这样一来，你就可以在不知道有序集的最低和最高 **score** 值的情况下，使用`ZCOUNT`、 `ZRANGEBYSCORE` 这类命令。

默认情况下，区间的取值使用**闭区间**(小于等于或大于等于)，你也可以通过给参数前增加 **(** 符号来使用可选的**开区间**(小于或大于)。

举个例子：

```
ZRANGEBYSCORE zset (1 5
```

返回所有符合条件 **1** **&lt;** **score** **&lt;=** **5** 的成员，而

```
ZRANGEBYSCORE zset (5 (10
```

则返回所有符合条件 **5** **&lt;** **score** **&lt;** **10** 的成员。

## <a id="rangeId">区间范围</a>

合法的 **min** 和 **max** 参数必须包含 **(** 或者 **[**， 其中 **(**  表示开区间， 而 **[** 则表示闭区间。

特殊值 **+** 和 **-** ，在 **min** 和 **max** 参数中具有特殊的意义， 其中 **+** 表示正无限， 而 **-** 表示负无限。 因此， 向一个所有成员的分值都相同的有序集合发送命令`ZRANGEBYLEX <zset> - +` ， 命令将返回有序集合中的所有元素。

## <a id="subscriptId">下标说明</a>

下标参数 **start** 和 **stop** 都以 **0** 为底，也就是说，以 **0** 表示有序集第一个成员，以 **1** 表示有序集第二个成员，以此类推。也可以使用负数下标，以 **-1** 表示最后一个成员， **-2** 表示倒数第二个成员，以此类推。

超出范围的下标并不会引起错误：

* 当 **start** 的值比有序集的最大下标还要大，或是 **start** **&gt;** **stop** 时， 返回一个空列表。
* 当 **stop** 参数的值比有序集的最大下标还要大，那么 Redis 将 **stop** 当作最大下标来处理。

## <a id ="dictSorderId">字典序</a>

**字典排序（lexicographical order）** 是一种对于随机变量形成序列的排序方法。其方法是，按照字母顺序，或者数字小大顺序，由小到大的形成序列。

在英文字典中，排列单词的顺序是先按照第一个字母以升序排列（即a、b、c……z 的顺序）；如果第一个字母一样，那么比较第二个、第三个乃至后面的字母。如果比到最后两个单词不一样长（比如，sigh 和 sight），那么把短者排在前。

通过这种方法，我们可以给本来不相关的单词强行规定出一个顺序。“单词”可以看作是“字母”的字符串，而把这一点推而广之就可以认为是给对应位置元素所属集合分别相同的各个有序多元组规定顺序。

比如说有一个随机变量X包含{1 2 3}三个数值。其字典排序就是：

```shell
{} {1} {1 2} {1 2 3} {2} {2 3} {3}
```

‍
