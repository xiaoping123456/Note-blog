# Redis



## 认识NoSql

NoSql : 即Not-Only Sql （非关系型数据库），作为关系型数据库的补充。

作用：应对基于海量用户和海量数据前提下的数据处理问题。

特点：1）非结构化 2）无关联的 3）非SQL语句

|          |                           SQL                           |                            NoSql                             |
| :------: | :-----------------------------------------------------: | :----------------------------------------------------------: |
| 数据结构 |                         结构化                          |                           非结构化                           |
| 数据关联 |                         关联的                          |                           无关联的                           |
| 查询方式 |                         SQL查询                         |                            非SQL                             |
| 事务特征 |                          ACID                           |                             BASE                             |
| 存储方式 |                          磁盘                           |                             内存                             |
|  扩展性  |                          垂直                           |                             水平                             |
| 使用场景 | 1）数据结构稳定 2）相关业务对数据安全性、一致性要求较高 | 1）数据结构不固定 2）对一致性、安全性要求不高 3）对性能要求高 |

## 认识Redis

Redis是一个基于内存的键值型NoSql数据库。

**redis一共16个库（0-15）,默认使用0号库，可以在配置文件中修改**

特征：

1. 键值型
2. 单线程：每个命令具备原子性
3. 低延迟，速度快（基于内存、IO多路复用、良好的编码）
4. 支持数据持久化
5. 支持主从集群、分片集群
6. 支持多语言客户端



Redis是一个key-value的数据库，key一般是String类型，不过value的类型多种多样

![image-20220714202438370](https://picgo111.oss-cn-beijing.aliyuncs.com/image-20220714202438370.png)

## 通用指令

通用指令是部分数据类型的，都可以使用的指令，常见的有：

keys : 查看符合模板的所有key，不建议在生产环境设备上使用

​			keys *   , keys a*

del : 删除一个指定的key  返回值是删除的数量

exists : 判断key是否存在  存在返回1，不存在返回0

expire : 给一个key设置有效期（单位是s），到期会自动删除 

TTL：查看一个KEY的剩余有效期（-1为永久有效）

## key的层级结构

Redis的key允许有多个单词形成层级结构，多个单词之间用':'隔开，格式如下：

`项目名:业务名:类型:id`

这个格式并非固定，也可以根据自己的需求来删除或添加词条。

`user`相关的 key -> `xiaoping:user:1`

在可视化工具中可以清晰的看到层级结构

![image-20220714205828903](https://picgo111.oss-cn-beijing.aliyuncs.com/image-20220714205828903.png)



## 5大基本类型

### String类型

String类型，也就是字符串类型，是Redis中最简单的存储类型。
其value是字符串，不过根据字符串的格式不同，又可以分为3类：

1. string：普通字符串
2. int：整数类型，可以做自增、自减操作
3. float：浮点类型，可以做自增、自减操作

不管是哪种格式，底层都是字节数组形式存储，只不过是编码方式不同。字符串类型的最大空间不能超过512m.

**常用指令**

1. SET：添加或者修改已经存在的一个String类型的键值对
2. GET：根据key获取String类型的value
3. MSET：批量添加多个String类型的键值对
4. MGET：根据多个key获取多个String类型的value

![image-20220714204245685](https://picgo111.oss-cn-beijing.aliyuncs.com/image-20220714204245685.png)

5. INCR：让一个整型的key自增1

6. INCRBY:让一个整型的key自增并指定步长，例如：incrby num 2 让num值自增2

7. INCRBYFLOAT：让一个浮点类型的数字自增并指定步长

8. SETNX：添加一个String类型的键值对，前提是这个key不存在，否则不执行

   ​				与 `set key value nx`效果一致

9. SETEX：添加一个String类型的键值对，并且指定有效期（单位是s）`setex key ttl value`

   ​				与`set key value ex ttl`效果一致

### Hash类型

Hash类型，也叫散列，其value是一个无序字典，类似于Java中的HashMap结构。
String结构是将对象序列化为JSON字符串后存储，当需要修改对象某个字段时很不方便：

![image-20220714210205015](https://picgo111.oss-cn-beijing.aliyuncs.com/image-20220714210205015.png)

Hash结构可以将对象中的每个字段独立存储，可以针对单个字段做CRUD：

![image-20220714210242262](https://picgo111.oss-cn-beijing.aliyuncs.com/image-20220714210242262.png)

Hash类型可以单独改变某个value的field的value

**常用指令**

1. HSET key field value：添加或者修改hash类型key的field的值

2. HGET key field：获取一个hash类型key的field的值

   ![image-20220714210711799](https://picgo111.oss-cn-beijing.aliyuncs.com/image-20220714210711799.png)

   ![image-20220714210725295](https://picgo111.oss-cn-beijing.aliyuncs.com/image-20220714210725295.png)

3. HMSET：批量添加多个hash类型key的field的值

4. HMGET：批量获取多个hash类型key的field的值

   ![image-20220714210851071](https://picgo111.oss-cn-beijing.aliyuncs.com/image-20220714210851071.png)

5. HGETALL：获取一个hash类型的key中的所有的field和value

   ![image-20220714210955999](https://picgo111.oss-cn-beijing.aliyuncs.com/image-20220714210955999.png)

6. HKEYS key：获取一个hash类型的key中的所有的field

7. HVALS key：获取一个hash类型的key中的所有的value

8. HINCRBY key field step:让一个hash类型key的字段值自增并指定步长

   ​											整型是hincrby ,浮点型是hincrbyfloat

9. HSETNX ：添加一个hash类型的key的field值，前提是这个field不存在，否则不执行

   ​					和`hset key field value nx `效果一致

### List类型

Redis中的List类型与Java中的LinkedList类似，可以看做是一个**双向链表**结构。既可以支持正向检索和也可以支持反向检索。

特征：1）有序 2）元素可重复 3）插入和删除块 4）查询速度一般

常用来存储一个有序数据，例如：朋友圈点赞列表，评论列表等。

**常用指令**

1. LPUSH key  element ... ：向列表左侧插入一个或多个元素

   ![image-20220714213151830](https://picgo111.oss-cn-beijing.aliyuncs.com/image-20220714213151830.png)

   ![image-20220714213240768](https://picgo111.oss-cn-beijing.aliyuncs.com/image-20220714213240768.png)

2. LPOP key：移除并返回列表左侧的第一个元素，没有则返回nil

   ![image-20220714213324929](https://picgo111.oss-cn-beijing.aliyuncs.com/image-20220714213324929.png)

3. RPUSH key  element ... ：向列表右侧插入一个或多个元素

4. RPOP key：移除并返回列表右侧的第一个元素

5. LRANGE key star end：返回一段角标范围内的所有元素 (从0开始)

6. BLPOP和BRPOP：与LPOP和RPOP类似，只不过在没有元素时等待指定时间，而不是直接返回nil

   blpop key timeout（timeout单位是s）  :    示例：张三执行了该语句 进入线程阻塞 只用另一个人在list中移除了一个元素，才可以解除阻塞。

   

- 如何利用List结构模拟一个栈？

​		ans : 入口和出口在同一边

- 如何利用List结构模拟一个队列？

​		ans : 入口和出口在不同边

- 如何利用List结构模拟一个阻塞队列？

​		ans : 入口和出口在不同边，出队采用blpop或brpop

### Set类型

Redis的Set结构与Java中的HashSet类似，可以看做是一个value为null的HashMap。因为也是一个hash表，因此具备与HashSet类似的特征：

1）无序 2）元素不可重复 3）查找快 4）支持交集、并集、差集等功能

**常用指令**

1. SADD key member ... ：向set中添加一个或多个元素
2. SREM key member ... : 移除set中的指定元素
3. SCARD key： 返回set中元素的个数
4. SISMEMBER key member：判断一个元素是否存在于set中
5. SMEMBERS：获取set中的所有元素
6. SINTER key1 key2 ... ：求key1与key2的交集

### SortedSet类型

Redis的SortedSet是一个可排序的set集合，与Java中的TreeSet有些类似，但底层数据结构却差别很大SortedSet中的每一个元素都带有一个score属性，可以基于score属性对元素排序，底层的实现是一个跳表（SkipList）加 hash表。

特性：1）可排序 2）元素不重复 3）查询速度快

因为SortedSet的可排序特性，经常被用来实现排行榜这样的功能。

**常用指令**

1. ZADD key score member：添加一个或多个元素到sorted set ，如果已经存在则更新其score值
2. ZREM key member：删除sorted set中的一个指定元素
3. ZSCORE key member : 获取sorted set中的指定元素的score值
4. ZRANK key member：获取sorted set 中的指定元素的排名
5. CARD key：获取sorted set中的元素个数
6. ZCOUNT key min max：统计score值在给定范围内的所有元素的个数
7. ZINCRBY key increment member：让sorted set中的指定元素自增，步长为指定的increment值
8. ZRANGE key min max：按照score排序后，获取指定排名范围内的元素 
9. ZRANGEBYSCORE key min max：按照score排序后，获取指定score范围内的元素
10. ZDIFF、ZINTER、ZUNION：求差集、交集、并集

**注**：

1. 所有的排名默认都是升序，如果要降序则在命令的Z后面添加REV即可

		2. 排名从0开始
		2. 范围都是 左右均是闭区间

## Redis的Java客户端

![image-20220714221614881](https://picgo111.oss-cn-beijing.aliyuncs.com/image-20220714221614881.png)

### Jedis

#### 使用步骤

1. 引入依赖

   ```xml
   <dependency>
   	<groupId>redis.clients</groupId>
   	<artifactId>jedis</artifactId>
   	<version>4.2.0</version>
   </dependency>
   ```

2. 创建Jedis对象，建立连接

   ```java
   private Jedis jedis;
   
   @BeforeEach
   void setUp(){
       //1.建立连接
       jedis = new Jedis("192.168.238.130",6379);
       //2.设置密码
       jedis.auth("123456");
       //3.选择库
       jedis.select(0);
   }
   ```

3. 使用Jedis对象操作redis

   ```java
   @Test
   void testString(){
       //存入数据
       String result = jedis.set("name","zs");
       System.out.println(result);
       //读取数据
       String name = jedis.get("name");
       System.out.println(name);
   }
   
   @Test
   void testHash(){
       jedis.hset("user:1","name","jack");
       jedis.hset("user:1","age","18");
       Map<String, String> map = jedis.hgetAll("user:1");
       System.out.println(map);
   }
   ```

#### Jedis连接池

Jedis本身是线程不安全的，并且频繁的创建和销毁连接会有性能损耗，因此我们推荐大家使用Jedis连接池代替Jedis的直连方式。

创建Jedis连接池

```java
public class JedisConnectionFactory {

    private static final JedisPool jedispool;

    static {
        //配置连接池
        JedisPoolConfig poolConfig = new JedisPoolConfig();
        poolConfig.setMaxTotal(8);//最大连接数
        poolConfig.setMaxIdle(8);//最大空闲连接
        poolConfig.setMinIdle(0);//最小空闲连接
        poolConfig.setMaxWaitMillis(10000);//最大等待时间
        //创建连接池对象
        jedispool = new JedisPool(poolConfig,"192.168.238.130",6379,1000,"123123");
    }

    public static Jedis getJedis(){
        //获取连接池中的Jedis对象
        return jedispool.getResource();
    }

}
```

