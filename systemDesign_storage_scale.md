# 为什么要分析 QPS?:
* QPS 的大小决定了数据存储系统的选择
   * 一个读多写少的系统，**一定要使用 Cache 进行优化**
   * **一定要先分析 读多写少还是读少写多**
      * 面向用户的 读多写少 -- 从QPS的角度来说，一台 MySQL 就可以搞定了
      * 面向机器的 读少写多 -- 可以使用 Memcached 进行读操作优化 （__redis 读写都很快！__）eg. 爬虫
   * 如何优化query
# cache 的存在形式
    * memory
       1. 是相对于filesystem而言的
    * filesystem
       1. 是相对于network drive而言的 本地访问or远程访问
# cache 的类型
    1. LFU
    2. LRU
# cache 操作
1. 当需要update时 cache和db肿么一个操作顺序？-》先delete cache key 再set database key 
   1. 要考虑的是 
      * 两个操作的对象是2套系统 
      * 用户会希望看到什么behaviour？ -- 怎样不会让用户读到脏数据 怎样可以保证落案了？
      * groundtruth是什么 -- db
    4. eventual consistant
       * 定义：使用ttl 保证终究一致
# cache usecase
* auth service
  * db 然后也cache做一下优化

# memcached vs. redis
* memcached
  * 适合读多写少
  * cache-aside
   * 用户需要自己管理cache miss时的数据loading
   * 常见pattern：webserver与db和cache分别沟通 db与cache不直接沟通 eg. memcached + mysql<img width="691" alt="Screenshot 2022-03-24 at 17 28 22" src="https://user-images.githubusercontent.com/83515400/159964268-22496b44-8477-47e2-bc00-9c0d17fe7356.png">

* redis
  * 读写都很快
  * cache-through 
    * Cache负责和DB沟通，把数据持久化
    * 常见pattern：redis<img width="796" alt="Screenshot 2022-03-24 at 17 31 21" src="https://user-images.githubusercontent.com/83515400/159964830-51e28a65-fe3e-445e-bb5b-b3ddc42d01d2.png">
* 业内选择： memcached + sql
  * 比redis稳定 扩展性也更好

# 数据库选择标准
1. 大部分的情况，
   * 用SQL也好，
     * 行式存储
   * 用NoSQL也好，
     * 列式存储 事实上更细：
       * eg. cassandra:
         * 一条数据一般以 grid 为单位，事实上是`3`元数据：  row_key + column_key + value = 一条数据
         * value是一个serialization打包<img width="662" alt="Screenshot 2022-03-24 at 18 31 24" src="https://user-images.githubusercontent.com/83515400/159975913-03f59bcc-a9f0-401f-b9f0-b525e0c6dc13.png">
         * Cassandra 支持这样的“范围查询”: query(row_key, column_start, column_end)
         * row_key
           * 分布式的基本 表示在哪台机器上 用hash值 是无法进行range query的
           * 是我们所说key-value的key
         * column_key
           * 是可以做range query的 是用来做排序的
           * 可以是复合值
         * value
           * 一般来说是string 要么需要自己serialize一下
           * 用binary tree serialization
         * query时
           * row_key + column_key + value
         * insert时
           * row_key + column_start + column_end
         * 如果把friendship储存在cassandra 应该肿么设计？
           * how to store really depends on how to query -- what is the requirement? in this case, if A and B are friends
           * 可以只指定row_key不指定column_key 获得所有呀
           * row_key : uid
           * column_key : other uid -- 是用来filter的target
           * value: 存一些metadata好了呀 
         * 如果要存newsfeed咋存？
           * row_key: uid
           * column_key: compound <createTS, tweetid>
           * value: tweet_data
   * 都是可以的
3. 需要支持Transaction的话
   * **不**能选NoSQL
4. 你想不想偷懒很大程度决定了选什么数据库
   * SQL更成熟帮你做了很多事儿 
   * NoSQL很多事儿都要亲力亲为(Serialization, Secondary Index)
   * 数据库如何建index的？b+tree
5. 如果想省点服务器获得更高的性能
   * NoSQL就更好 
   * 硬盘型的NoSQL比SQL一般都要快10倍以上
6. 如果存朋友关系 friendship 应该肿么选择
   * 老公司facebook可能会用graphDB
   * 新公司会用cassandra

7. Scale: how to scale?
   * 要考虑QPS（影响storage选择）和 single point failure
   * single point failure
     * sharding
       * 把数据分布在不同机器上 依据什么来分布数据？ `sharding key` AKA `row key` AKA `partition key`
       * rationale：不会100%不可用
       * SQL
         * does **NOT** come with sharding!!! need to implement your own!
       * NoSQL
         * 大多自带sharding -- 程序猿开发nosql的原因。。。
         * eg Cassandra 只要告诉我你要读什么 我帮你觉得它在哪台机器上 帮你写好让你读好
       * sharding strategy
         * vertical sharding
           * 本质上按`column`拆分
           * eg Cassandra 不同table放不同机器 或者按变动频率拆分一个table 分成不同新的table 再放在不同机器上
           * 啥问题？
             * 还是要挂一起挂 一个功能
         * horizontal sharding
           * 本质上按`row`拆分
           * eg 按`uid mod 10`拆分
           * 啥问题：
             * hotspot 大v！-- 其实还好啦
             * migration！如果之前`10`台机器不够用了 变成`11`太 那要重新计算啊亲！ -- **大问题**
               * migraine！ 慢
               * 压力大易挂
               * 数据不一致
               * 很多奇怪的问题啊
             * 肿么办？-- 铛铛铛 **consistent hashing** ！
               * `mod`得大一点啊。。。  比如`360`
                 * 所以`mod`的结果没有变 变的只是机器的mapping关系 -- 这个区间分配关系存在web server上
                 * 所以只是挪动数据在机器间                                                                         
     * replication 
       * 一般重复三份
       * 且还可以读写分离
8. refs
   * cache:
     * Dynamo: Amazon’s Highly Available Key-value Store https://www.allthingsdistributed.com/files/amazon-dynamo-sosp2007.pdf
     * Scaling Memcache at Facebook https://www.usenix.org/system/files/conference/nsdi13/nsdi13-final170_update.pdf
   * consistent hashing:
     * Consistent hashing https://michaelnielsen.org/blog/consistent-hashing/
     * 一致性hash算法 - consistent hashing https://blog.csdn.net/sparkliang/article/details/5279393
   * others:
     * Coach Base Architecture: http://horicky.blogspot.com/2012/07/couchbase-architecture.html
