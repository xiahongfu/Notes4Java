# 过期策略
```c
/* Redis database representation. There are multiple databases identified
 * by integers from 0 (the default database) up to the max configured
 * database. The database number is the 'id' field in the structure. */
typedef struct redisDb {
    dict *dict;            // 存放key-value     /* The keyspace for this DB */
    dict *expires;         // 存放key-ttl     /* Timeout of keys with a timeout set */
    dict *blocking_keys;        /* Keys with clients waiting for data (BLPOP)*/
    dict *ready_keys;           /* Blocked keys that received a PUSH */
    dict *watched_keys;         /* WATCHED keys for MULTI/EXEC CAS */
    int id;                     /* Database ID */
    long long avg_ttl;          /* Average TTL, just for stats */
    unsigned long expires_cursor; /* Cursor of the active expire cycle. */
    list *defrag_later;         /* List of key names to attempt to defrag one by one, gradually. */
} redisDb;
```
redisDb使用expires为所有设置了超时时间的key记录key-ttl。ttl是unix时间戳，精确到毫秒。

### 过期回收策略
redis在回收key时不是过期了立马回收，而是采用惰性删除和定期删除结合的策略。
**惰性删除**：每次从数据库访问key的时候去检查是否过期，如果过期了则删除。
**定期删除**：Redis在初始化时会生成一个定时任务，周期性的抽样部分过期的key，然后进行删除。

定期删除有两种删除模式：
**FAST模式**：执行频率受beforeSleep()调用频率影响，但是两次FAST模式间隔不低于2ms，一次执行时间最长为1ms。执行过程如下：
逐个遍历db，逐个遍历db中的bucket，抽取20个key判断是否过期。
如果过期比例大于25%且不超过清理时间上限，则继续执行清理工作。

**SLOW模式**：执行频率受server.hz影响，默认为10，即每秒执行10次，每个执行周期100ms。一次slow清理耗时不超过执行周期的25%。执行过程如下：
逐个遍历db，逐个遍历db中的bucket，抽取20个key判断是否过期。
如果过期比例大于25%且不超过清理时间上限，则继续执行清理工作。

bucket是啥？redis数据库采用dict存储key-ttl键值对，bucket记录的是dictEntry的id。用来判断当前采样到了dict的什么位置。

**主从和AOF处理过期key**
过期后会在AOF中存入一个DEL命令，

# 内存淘汰
当redis内存使用达到设置的阈值时，主动挑选部分key删除以释放更多内存的过程。redis在**所有命令执行之前**进行内存检查。
以下是redis支持的8种内存淘汰策略：

* **noeviction**：不淘汰任何key，内存满时不允许写入新数据。默认策略
* **volatile-ttl**：对于设置了ttl的key，剩余时间越少越先被淘汰
* **allkeys-random**：对所有key，随机淘汰
* **volatile-random**：对设置了TTL的key，随机淘汰
* **allkeys-lru**：对所有key，采用LRU算法进行淘汰
* **volatile-lru**：对设置了TTL的key，采用LRU算法进行淘汰
* **allkeys-lfu**：对所有key，采用LFU算法进行淘汰
* **volatile-lfu**：对设置了TTL的key，采用LFU算法进行淘汰

**LRU(Least Recently Used)**：用当前时间减去最后一次访问时间，这个值越大越先被淘汰。
**LFU(Least Frequently Used)**：会统计key的访问频率，值越小越先被淘汰。

**如果使用LRU策略，怎么知道最后一次访问时间呢？**
robj中存放了一个`unsigned lru:LRU_BITS`，当使用LRU策略时，这里保存的就是24字节的上一次使用时间，单位为秒。
**如果使用LFU策略，redis是如何统计访问频率的？**
`unsigned lru:LRU_BITS`前16字节以分钟为单位记录最近一次访问时间，低8位记录逻辑访问次数。
**逻辑访问次数是怎么来的？**（近似计数算法Morris counter）
* 生成0~1之间的随机数R
* 计算1/(旧次数 * lfu_log_factor + 1)，记为P，lfu_log_factor默认为10
* 如果R<P，则计数器+1，且最大不超过255
* 访问次数会随着时间衰减，距离上一次访问时间每隔lfu_decay_time（默认为1）分钟，计数器-1

redis没有真正实现LRU，因为太消耗内存了。redis实现的是近似LRU算法，在满足特定条件的候选key之间进行LRU算法（LFU算法类似）。可以通过`maxmemory-samples 5`来控制LRU算法的精确度。
> Redis LRU algorithm is not an exact implementation. This means that Redis is not able to pick the best candidate for eviction, that is, the access that was accessed the furthest in the past. Instead it will try to run an approximation of the LRU algorithm, by sampling a small number of keys, and evicting the one that is the best (with the oldest access time) among the sampled keys.

# 参考资料
[EXPIRE](https://redis.io/commands/expire/)
[Key eviction](https://redis.io/docs/reference/eviction/)