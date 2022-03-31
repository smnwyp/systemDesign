* indexing
  * a way to reduce entropy
  * to store and to query 
  * db storage develops on top of b+ trees, with a twist:
    * one-piece pointer  to point the other leaf node 
* mysql storage engines
  *  <img width="943" alt="Screenshot 2022-03-31 at 13 16 47" src="https://user-images.githubusercontent.com/83515400/161043230-e6abef56-0598-45c3-b593-da62e0721e29.png">
  *  myisam
     * non-transactional 
     * non-clustered index  
       * addr and data are seperated
     * primary index is not required
     * leaf stores addr, need to look up 
  *  innodb
     * transactional  
     * clustered index
     * primary index is required
       * either innodb automatically chooses a unique column
       * or innodb creates a hidden one
       * very important!!
       * dont use very long str field as primary index
         * secondary index would be too big.. 
       * some monotonic ordering would be nice, dont use random ordering field!
         * otherwise b+ tree would rebalance too frequently
         * uuid is not the best idea
       * selectivity ~ 1 --> good candidate
       * <img width="811" alt="Screenshot 2022-03-31 at 13 33 09" src="https://user-images.githubusercontent.com/83515400/161045685-734dbde7-c4d3-4e90-8b29-e19effdee0a7.png">
       * 最好用跟业务无关的int 比如自增id
       * 
     * secondary index stores primary index value! need to search again using primary index 
       * does NOT need to be unique
* composite index!   
  * eg. vorname + nachname
  * <img width="973" alt="Screenshot 2022-03-31 at 13 15 59" src="https://user-images.githubusercontent.com/83515400/161043107-8cdc7b34-6ef8-4898-8897-9a0f1a197e79.png">
  * Composite Unique Index
    * unique index combination in the composite index 

* how to design
  * decide on primary index
  * decide on composite index
  * decide on unique index

* transaction
  * <img width="891" alt="Screenshot 2022-03-31 at 16 09 39" src="https://user-images.githubusercontent.com/83515400/161075299-5c24cebc-c60f-4432-979b-e4b3d1f20675.png">
  * consistency is the key!!
    * how to ensure
      * Concurrency Control
        * transaction Isolation to ensure consistency
        * <img width="849" alt="Screenshot 2022-03-31 at 14 09 06" src="https://user-images.githubusercontent.com/83515400/161051472-27c5332a-362d-4cf3-8343-fc43e01b0573.png">
      *  Log Recovery
        * to ensure 原子性(Atomicity) and 持久性 (Durability) 
  * isolation levels:
    *   <img width="922" alt="Screenshot 2022-03-31 at 13 54 08" src="https://user-images.githubusercontent.com/83515400/161049088-ae44ba4e-4575-4b2b-b452-32610aa0dd46.png">
  * concurrency control
    * pesimistic
      * locks
        * <img width="1201" alt="Screenshot 2022-03-31 at 15 58 13" src="https://user-images.githubusercontent.com/83515400/161072779-4c25ce7b-f569-4c4a-b948-3ca247a06f4d.png">
      * timestamp
        *   对于可能造成冲突的任何并发操作，基于时间戳排序规则，选定某事务继续执行，其他事务 回滚(Rollback)。
        *   <img width="811" alt="Screenshot 2022-03-31 at 15 57 55" src="https://user-images.githubusercontent.com/83515400/161072707-e1c5034e-f4c6-4d0c-8460-a11730984a30.png">
        *   
    * optimistic
      * 有效性检查
        * 事务对数据的更新首先在自己的工作空间进行，等到要写回数据库时才进行有效性检查，对不符合要求的事务进行回滚。
        * 对事务的时间戳进行比较完成的。 允许可能冲突的操作并发执行，因为每个事务操作的都是自己工作空间(Workspace)的局部变量。 直到有效性检查阶段发现了冲突才回滚。
        * 读阶段
          * 数据项被读入并保存在事务的局部变量(Local Variable)中。所有后续 写操作都是对局部变量进行，并不对数据库进行真正的更新。
        * 有效性检查阶段 
          * 对事务进行有效性检查，判断是否可以执行写操作而不违反可串行性(Serializable)。如果失败，则回滚该事务。 
        * 写阶段
          * 事务已通过有效性检查，将临时变量中的结果更新到数据库中。
* 数据库故障恢复
  * 原子性(Atomicity)、持久性(Durability) 和 一致性(Consistency) 保障
  * 故障类别
    * 数据库系统故障
      * 由于软件问题、硬件错误或者操作系统异常，导致数据库系统崩溃或终止。 
    * 事务故障
      * 由于非法输入或者数据库出现死锁(Deadlock)等，导致事务无法继续执行。
      * 故障会对事务、数据库状态(如数据库存储的数据)造成损坏。 需要对故障进行恢复，以保证数据库一致性(Consistency)，事务的原子性(Atomicity)以及持久性 (Durability)。
      * 以日志(Write Ahead Log, WAL)的方式记录对数据库的操作。 在故障时根据日志进行恢复，称为**日志恢复技术**。
        * <img width="1056" alt="Screenshot 2022-03-31 at 16 09 06" src="https://user-images.githubusercontent.com/83515400/161075103-d73f76c1-4c1e-4f63-b0ee-feddcbc6826a.png"> 
   * 事务执行过程中的故障
      * 立即修改
        * 数据库在事务提交前出现故障，但是事务的部分修改已经写入磁盘中。这破坏了事务的原子性(Atomicity)。 
        * 用日志方式解决
          *   
      * 延迟修改
        * 数据库在事务提交后出现故障，但数据还在内存缓冲区(Buffer)中，未写入磁盘。系统恢复时将丢失此次已提交的修改。这破坏了事务的持久性(Durability)。
        * 数据库的延迟修改
          * 1. 数据库系统为每个事务分配一个私有的工作区
          * 2. 事务的读操作从磁盘中拷贝数据项到工作区中。执行写操作前，仅操作工作区内数据的拷贝。
          * 3. 事务的写操作把数据输出到内存的缓冲区中。等到合适的时机，数据库的缓冲区管理器将数据写入到磁盘。  
        * 用日志方式解决
          * 
     
