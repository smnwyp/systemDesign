* 4s
  * 通过scenario分析 获得读写功能的评估 
  * 通过service分析 land on master-client模式 
  * storage具体肿么master-slave存
  * -- 以上3s 可以获得working solution
  * scale是**升级优化**
* storage
  * store
    * 普通文件系统 Meta Data，Block
    * 大文件存储: Block-> Chunk
    * 多台机器超大文件: Chunk Server + Master
  * 写入
    * Master+Client+ChunkServer 沟通流程
    * Master 维护metadata 和 chunkserver 表
  * 读出
    * Master+Client+ChunkServer 沟通流程
* large file storage
  * type
    * p2p
    * master-slave
      * meta data
        * `1 chunk` = `64MB` needs `64B`. (经验值)  (10^6 系数)
        * `10P=16*10^6` chunk needs `10G`
      * master的job
        * 存储各个文件数据的metadata
        * 存储Map(file name + chunk index -> chunk server)
          * **读取** 时找到对应的chunkserver
          * **写入** 时分配空闲的chunkserver
* usecase：大文件多次写入or？
  * 当然多次写入啦
  * 每份多大呢？
    *  文件本来就是按chunk存的 那就按chunk传咯
  * 谁来分？master还是client？
    * client 把文件拆分为n份，每一份一个chunk index
      * 比如 /gfs/home/dengchao.mp4 size = 576M. 那么可以切分问 576M/64M = 9个chunk
  * 肿么传输？
    * client先跟master沟通要写哪个chunk，然后master分配chunkserver--client就知道要把对应chunk写入哪个server
    * client写完告诉master写完了
  * 肿么修改？
    * 先删除
    * 重新把整个文件重写一份
    * 只写不改！高效快速呀
  * 肿么读？
    *  client从master获得chunk meta
    *  然后从不同的chunkserver读取
* 如何优化
  * 单master？ -- 工业界90%的系统都采用单master
    * 双master： http://borthakur.com/ftp/RealtimeHadoopSigmod2011.pdf Apache Hadoop Goes Realtime at Facebook Multi Master
    * 多master： Paper: Paxos Algorithm https://www.quora.com/In-distributed-systems-what-is-a-simple-explanation-of-the-Paxos-algorithm
  * failure and recovery？
    * google的便宜机器 坏了咋办 咋detect？
      *  CheckSum Method (MD5, SHA1, SHA256 and SHA512)
         * 一位多位 不同算法
         * 边写边加在chunk里
           * 1 checksum size? -- 4bytes = 32bit
           * 1 chunk = 64MB 
           * Each chunk has a checksum
           * The size of checksum of `1P` file • `1P/64MB*32`bit = `62.5` MB  
         * 何时检查checksum
           * when read
             * 重新读数据并且计算现在的checksum
             * 比较现在的checksum和之前存的checksum是否一样
           * periodically
     * How to avoid chunk data loss when a ChunkServer is down/fail?
       * Replica
         * 存3份够了。。。
         * 放在不同地方！2近1远
         * 选chunk server备份的时候有什么策略?   
           * 最近写入比较少的 (LRU)
           * 硬盘存储比较低的
     * 如何修复 How to recover when a chunk is broken?
       * Ask master for help
       * adsf
     * 肿么检测机器活着 How to find whether a ChunkServer is down?
       * monitoring呀 HeartBeat
         * chunkservers->master
         * master只要监听即可
         * master一般管1k-2k机器
   * 有replica时如何scale write？
     *  以前写一台 现在要写好几台 client写会成为瓶颈咩
     *  那就写完一个chunkserver后 让它populate 其他的chunkservers (through **leader election**)
        * 找距离最近的(快）
        * 找现在不干活的(平衡traffic)
     *  如果chunkserver挂了咋办
        * lead chunkserver挂了
          * client要retry -- master分配个新的机器
        * follower chunkserver挂了

* hdfs vs. gfs
  * 基本等价
  * name node -> master
  * data node -> chunkserver


* Key Point: 
  * Master-Slave
  * Storage:
    * Save a file in one machine -> a big file in one machine -> a extra big file in multi-machine
    * Multi-machine
      * How to use the master?
      * How to traffic and storage of master?
  * Read
    * The process of reading a file
  • Write:
    * The process of writing a file
    * How to reduce master traffic
      * Client 和 Chunk Server沟通 
    * How to reduce client traffic?
      * Leader Election
  * Failure and Recover (key)
    * Discover the failure a chunk? 
      * Check Sum
    * Avoid the failure a chunk? 
      * Replica
    * Recover the failure?
    * Ask master
    * Discover the failure of the chunkserver? 
      * Heart Beat
    * Solve the failure of writing ChunkServer? 
      * Retry
