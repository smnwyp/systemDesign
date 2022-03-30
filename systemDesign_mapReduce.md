* 如何使用MapReduce -- 非常重要
* 要考虑的点：
  * **map的output** `key:val` 是啥
  * **reduce的output** `key:val` 是啥 
* process flow:
  1. MapReduce is Master-Slave architecture
  2. **start** user defines `no.map` `no.slave`, and starts `all servers (slaves and master)`
  3. **assign** master decides which machines shall be `map servers`, and which machines shall be `reduce servers`
  4. **split** master splits up the input data as evenly as possible 
  5. **map read** each `map servers` reads input data
  6. **map** each `map server` does map
  7. **map output** each map server will write its output to its hard-disk
  8. **reduce fetch** `reduce servers` fetch data from `map servers` to aggregate
  9. **reduce** `reduce servers` will start aggregating results
  10. **reduce output** `reduce servers` will output final data 
* questions:
  * what happens if servers fail?
    * there will be backup servers
  * what if certain `reduce keys` are super numerous?
    * random postfix -- salting
      * extra script to combine
  * where does one store the inputs and outputs?
    * GFS?
    * do we also store results from `map` in GFS instead of just local server?
      * no, just redo when lost
      * intermediate results do not need to be stored
      * storage over network is slower than storing locally
  * can `map` and `reduce` be carried out on the same server?
    * the inherent features like fault-tolerance are lost
    * when the data is small and the computing is less, then Hadoop has a lot overheads compared to the actual processing.
    *    
