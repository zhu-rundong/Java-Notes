## 分布式锁的特征
1.  **互斥性：** 任何时刻有且仅有一个线程持有锁
2.  **超时释放：** 必须有超时控制机制，防止死锁
3.  **重入性：** 同一个节点的同一个线程如果获得锁之后，它可以再次获取这个锁
4.  **高可用：** 不能因为某一个节点挂了而出现获取锁和释放锁失败的情况
5.  **安全性：** 自己加锁自己释放，锁只能被持有该锁的客户端解锁，不能有其他客户端解锁

## 方案一：SETNX + EXPIRE
使用 `setnx` 来抢锁，抢到之后，再使用 `expire` 给锁设置一个过期时间，防止忘记释放锁。
```java
@Autowired  
private StringRedisTemplate stringRedisTemplate;

public String test(){
    String KEY = "distributedLock";
    String value = UUID.randomUUID().toString()+Thread.currentThread().getName();
    try {
        //抢锁
        Boolean flagLock = stringRedisTemplate.opsForValue().setIfAbsent(key, value);
        //设置过期时间
        stringRedisTemplate.expire(key,10L,TimeUnit.SECONDS);

        if(!flagLock){
            return "抢锁失败";
        }
        //TODO 业务逻辑
            
    }finally {
        stringRedisTemplate.delete(key);
    }
}        
```
但是这个方案存在一个问题，`setnx` 和 `expire` 两个命令不是原子性。如果执行完 `setnx`，服务器宕机了，那么，这个锁就永远不会释放了。

## 方案二：SET key value [EX seconds]  [PX milliseconds]  [NX|XX]
```java
@Autowired  
private StringRedisTemplate stringRedisTemplate;

public String test(){
    String KEY = "distributedLock";
    String value = UUID.randomUUID().toString()+Thread.currentThread().getName();
    try {
        //抢锁
         Boolean flagLock = stringRedisTemplate.opsForValue()
             .setIfAbsent(key,value,10L,TimeUnit.SECONDS);

        if(!flagLock){
            return "抢锁失败";
        }
        //TODO 业务逻辑
            
    }finally {
        stringRedisTemplate.delete(key);
    }
}        
```
这个方案仍然存在一个问题：**误删锁。**
例如，线程 A 在执行业务逻辑时，用时超过 10 秒钟，锁自动过期了。这时，线程 B 进来，重新获取了该锁，但是，当线程 A 比线程 B 先执行到了 `finally`，执行了锁删除操作，删除的却是线程 B 的锁，这就造成了误删锁的情况。

## 方案三：SET key value [EX seconds] [PX milliseconds] [NX|XX] + 随机数检验
```java
@Autowired  
private StringRedisTemplate stringRedisTemplate;

public String test(){
    String KEY = "distributedLock";
    String value = UUID.randomUUID().toString()+Thread.currentThread().getName();
    try {
        //抢锁
         Boolean flagLock = stringRedisTemplate.opsForValue()
             .setIfAbsent(key,value,10L,TimeUnit.SECONDS);

        if(!flagLock){
            return "抢锁失败";
        }
        //TODO 业务逻辑
            
    }finally {
        if (stringRedisTemplate.opsForValue().get(key).equals(value)) {
            stringRedisTemplate.delete(key);
        }
    }
}        
```
但是，这个方案仍有问题， `finally` 中锁的判断和删除不是原子性 。

## 方案四：使用 Lua 脚本
```java
@Autowired  
private StringRedisTemplate stringRedisTemplate;

public String test(){
    String KEY = "distributedLock";
    String value = UUID.randomUUID().toString()+Thread.currentThread().getName();
    try {
        //抢锁
         Boolean flagLock = stringRedisTemplate.opsForValue()
             .setIfAbsent(key,value,10L,TimeUnit.SECONDS);

        if(!flagLock){
            return "抢锁失败";
        }
        //TODO 业务逻辑
            
    }finally {
        Jedis jedis = RedisUtils.getJedis();

        String script = "if redis.call('get', KEYS[1]) == ARGV[1] " +
                "then " +
                "return redis.call('del', KEYS[1]) " +
                "else " +
                "   return 0 " +
                "end";

            try {
                Object result = jedis.eval(script
                , Collections.singletonList(REDIS_LOCK_KEY)
                , Collections.singletonList(value));
                if ("1".equals(result.toString())) {
                    System.out.println("------>Delete DistributedLock success");
                }else{
                    System.out.println("------>Delete DistributedLock error");
                }
            } finally {
                if(null != jedis) {
                    jedis.close();
                }
            }

    }
}        
```

**至此，基于单个 Redis 节点实现分布式锁告一段落。但是，还有一个问题，如何确保 redisLock 过期时间大于业务执行时间的问题？锁如何续期？**

当线程获得锁时，给获取线程的锁开启一个定时守护线程，每隔一段时间检查锁是否还存在，存在则对锁的过期时间延长，防止锁过期提前释放，这就是所谓 `watch dog` 看门狗。

## 方案五：Redisson
Redis 一般都是集群部署，主从的异步复制可能会造成的锁丢失。例如：线程 A 在 master 获取锁，但 master 还没来的及把刚刚加锁的 key 同步到 slave 点，master 就挂了，slave 变成了 master，线程 B 就可以获取同个 key 的锁，不符合分布式锁的互斥性特征。

Redis 作者 antirez 提出了一种分布式锁算法：`Redlock`。**Redisson** 就是 `Redlock` 的 Java 版实现。

```java
@Autowired  
private StringRedisTemplate stringRedisTemplate;
@Autowired  
private Redisson redisson;
public static final String KEY = "distributedLock";
public String test(){
    //抢锁
    RLock redissonLock = redisson.getLock(KEY);  
    redissonLock.lock();
    try {
        //TODO 业务逻辑
            
    }finally {
        if(redissonLock.isLocked() && redissonLock.isHeldByCurrentThread())  {  
            redissonLock.unlock();  
        }
    }
}        
```
