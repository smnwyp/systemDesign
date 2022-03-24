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
           * 是可以做range query的 
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
 


 
