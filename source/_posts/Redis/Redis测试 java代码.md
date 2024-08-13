---
title: Redis测试-java代码
date: 2018-06-04 23:41
tags: 
  - Redis
categories:
  - [Redis]
---

pom.xml
```
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>2.9.0</version>
</dependency>
```

## 简单测试

```
public static void main(String[] args) {
    Jedis jedis = new Jedis("192.168.1.213");
    jedis.set("foo", "bar");
    String value = jedis.get("foo");
    System.out.println(value);
}
```
如果用这种方法连集群，会看到如下的错误
```
Exception in thread "main" redis.clients.jedis.exceptions.JedisMovedDataException: MOVED 12182 192.168.1.214:6382
```

## Redis集群测试

```
public static void main(String[] args) {
    Set<HostAndPort> jedisClusterNodes = new HashSet<HostAndPort>();
    // Jedis集群将尝试自动发现集群节点
    jedisClusterNodes.add(new HostAndPort("192.168.1.214", 6379));
    // jedisClusterNodes.add(new HostAndPort("192.168.1.214", 6380));
    // jedisClusterNodes.add(new HostAndPort("192.168.1.214", 6381));
    JedisCluster jc = new JedisCluster(jedisClusterNodes);
    jc.set("foo", "bar");
    jc.set("foo1", "bar1");
    jc.set("foo2", "bar2");
    jc.set("foo3", "bar3");
    jc.set("foo4", "bar4");
    jc.set("foo5", "bar5");
    jc.set("foo6", "bar6");

    System.out.println(jc.get("foo"));
    System.out.println(jc.get("foo1"));
    System.out.println(jc.get("foo2"));
    System.out.println(jc.get("foo3"));
    System.out.println(jc.get("foo4"));
    System.out.println(jc.get("foo5"));
    System.out.println(jc.get("foo6"));

    System.out.println("===================");

    Map<String, JedisPool> map = jc.getClusterNodes();
    for (Map.Entry<String, JedisPool> e : map.entrySet()) {
        System.out.println(e.getKey());
        System.out.println("===================");
    }

}
```

运行时报错：
```
Exception in thread "main" redis.clients.jedis.exceptions.JedisConnectionException: Could not get a resource from the pool
	at redis.clients.util.Pool.getResource(Pool.java:53)
	at redis.clients.jedis.JedisPool.getResource(JedisPool.java:226)
	at redis.clients.jedis.JedisSlotBasedConnectionHandler.getConnectionFromSlot(JedisSlotBasedConnectionHandler.java:66)
...
...
```

因为在之前创建创建集群时使用的IP是127.0.0.1，当返回(error) MOVED 10439 127.0.0.1:6381时，系统尝试连接本地的端口，然后连不上失败了

修改Redis配置文件，绑定到正确的IP上，然后重新创建集群指定具体的IP地址：

```
./redis-trib.rb create --replicas 1 192.168.1.214:6379 192.168.1.214:6381 192.168.1.214:6382 192.168.1.214:6383 192.168.1.214:6384 192.168.1.214:6385
```

----
```
#杀掉redis进程
ps -ef | grep redis | awk '{print $2}' | xargs kill

#删除
rm -f /data/redis/d63*/dump.rdb
rm -f /data/redis/d63*/nodes.conf

#批量替换
sed -i 's/192.168.1.214/0.0.0.0/' */redis.conf

```

## Jedis客户端分片测试
分片使用一种称为“一致哈希”的技术，并根据一些散列算法（md5和murmur，后者不太标准，但速度更快）在一组redis服务器上同等地分配key值。这样的节点被称为“分片”。优点是每个分片只会占总内存的 1/n（对于n是分片数量）。

>先配置三个独立的redis服务 

### 直连接方式
```
public static void main(String[] args) {
    List<JedisShardInfo> shards = new ArrayList<JedisShardInfo>();
    JedisShardInfo si = new JedisShardInfo("192.168.1.213", 6380);
    shards.add(si);
    si = new JedisShardInfo("192.168.1.213", 6381);
    shards.add(si);
    si = new JedisShardInfo("192.168.1.213", 6382);
    shards.add(si);

    // 直接连接
    ShardedJedis jedis = new ShardedJedis(shards);
    jedis.set("a", "foo");
    jedis.set("a1", "foo1");
    jedis.set("a2", "foo2");
    jedis.set("a3", "foo3");
    jedis.set("a4", "foo4");

    System.out.println(jedis.get("a"));
    System.out.println(jedis.get("a1"));
    System.out.println(jedis.get("a2"));
    System.out.println(jedis.get("a3"));
    System.out.println(jedis.get("a4"));

    ShardInfo<?> sinfo = jedis.getShardInfo("a");
    System.out.println(sinfo);

    jedis.disconnect();
    jedis.close();
}
```
### 连接池方式
```
public static void main(String[] args) {
    List<JedisShardInfo> shards = new ArrayList<JedisShardInfo>();
    JedisShardInfo si = new JedisShardInfo("192.168.1.213", 6380);
    shards.add(si);
    si = new JedisShardInfo("192.168.1.213", 6381);
    shards.add(si);
    si = new JedisShardInfo("192.168.1.213", 6382);
    shards.add(si);

    // 池连接
    JedisPoolConfig jedisPoolConfig = new JedisPoolConfig();
    ShardedJedisPool pool = new ShardedJedisPool(jedisPoolConfig, shards);

    try (ShardedJedis jedis = pool.getResource()) {
        jedis.set("x", "foo");
        jedis.set("x1", "foo1");
        jedis.set("x2", "foo2");
    }

    try (ShardedJedis jedis2 = pool.getResource()) {
        jedis2.set("y", "bar");
        jedis2.set("y1", "bar1");
        jedis2.set("y2", "bar2");
    }

    try (ShardedJedis jedis = pool.getResource()) {
        System.out.println(jedis.get("x"));
        System.out.println(jedis.get("x1"));
        System.out.println(jedis.get("x2"));
    }

    try (ShardedJedis jedis2 = pool.getResource()) {
        System.out.println(jedis2.get("y"));
        System.out.println(jedis2.get("y1"));
        System.out.println(jedis2.get("y2"));
    }

    pool.close();
}
```

