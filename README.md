# MongodbTree
Mongo 文档存储数据库

![](https://i.imgur.com/fXnMvOA.png)

<pre>
Mongodb技术
1）数据类型
   mongo的文档格式 BSON, 二进制的JSON
2）_id, ObjectId
   
3) 索引
     1）唯一索引
        唯一索引确保集合中的每一个文档的指定键都是唯一的
           db.ponapartetable.ensureIndex({"uid":1}, {"unique":true})
           典型的唯一索引"_id",在集合创建的时候自动创建，"_id"唯一索引不能被删除，其他唯一索引都是可以删除的 

     2）稀疏索引
           唯一索引会把null看做值，所以无法将多个缺少唯一索引中的键的文档插入到集合中，然而，有些情况下，你可能希望唯一索引只对
           包含相应键的文档生效，如果有一个可能存在也可能不存在的字段，但是当它存在时，它必须是唯一的，这时就可以将unique和sparse选项组合在一起使用
           db.ponapartetable.ensureIndex({"uid":1}, {"unique":true, "sparse":true})
           
          hint()强制进行全表扫描

     3) 所有的数据库索引信息全部放在system.indexes集合中，保留集合
     5) 默认情况下，Mongodb会尽可能快的创建索引，阻塞所有对数据库的读请求和写请求，一直到索引创建成功，如果希望在创建索引时仍能处理请求，可通过创建索引时指定backgroud选项，
     有请求需要处理，创建索引会暂停一下，后台创建索引比前台创建索引慢很多
5）特殊的索引与集合
     普通集合:
	     1）动态创建的
		 2）可以自动增长以容纳更多的数据
     特殊集合:
         1）预先创建好
         2）大小固定
         固定集合的行类似于循环队列，如果已经没有空间了，最老的文档会被删除用以释放空间，新插入的文档会占据这块空间.
         固定集合的访问模式与mongodb中的大部分集合不太一样：数据被顺序写入磁盘上的固定空间，因为它们在磁盘上的访问速度非常快，固定集合可以用来记录日志
         创建固定集合：
            db.createCollection("myCollection", {"capped":true, "size":1000, "max":100})
	           集合大小为1000字节，单个文档最大为100
	        db.runCommand({"convertToCapped":"ponaparte", "size":10000})
	        将ponaparte转换为固定集合，无法将固定集合转化为非固定集合
         自然排序：
            对固定集合可以进行一种特殊的排序，称为自然排序，自然排序返回的结果就是集合中文档在磁盘上的顺序 
         创建不带"_id"索引的集合， createCollection({"autoIndexId":false})
	  TTL索引：
		 这种索引为每一个文档设置一个超时时间，一个文档达到预设值的老化时间之后就会被删除，这种索引对于缓存问题非常有用
         db.ponaparte.ensureIndex({"utime":1}, {"expireAfterSecs":60 * 60 * 24})
         在utime上建立TTL索引，24小时之后失效
      全文本索引
         使用正则表达式搜索大块文本存在的问题
         1）速度非常慢
	     2）无法处理语言的额问题，比如entry, entries应该算是匹配的
      使用全文本索引可以非常快的进行文本搜索，就像内置了多种语言的分词机制的支持一样
         1）创建任何一种索引的开销都比较大，创建全文本索引的成本更高
         2）全文本索引也会导致比普通索引更为严重的性能问题，因为所有字符串都需要被分解，分词，并且保存到一个地方，因此可能会发现全文本索引的集合的写入速度比其他集合要查，
         全文本索引也会降低分片时的数据迁移速度，将数据迁移到其他分片是，所有文本都需要重新进行索引
           启用全文本索引
              db.adminCommand({"setParameter":1, "textSearchEnabled":1})
         3）全文本索引只会对字符串数据进行索引，其他数据类型会被忽略	 
         5）一个集合中最多只能有一个全文本索引
         6）全文本索引与普通索引不同的是：全文索引中索引字段的顺序不重要，重要的是索引字段的权重，一经创建，权重不可修改，可以通过删除索引方式修改权重
      地理空间索引
         2dsphere索引（地球表面数据）
	        数据格式：GeoJson格式（指定点（longtitude, latitude），线，所变形）
	     2d索引（平面地图）	
      GridFS存储文件
               
6）聚合
      常用的聚合工具
         1）聚合框架
            使用聚合框架可以对集合中的文档进行变换和组合，基本上，可用多个构件创建一个管道(pipeline)，用于对一连串的文档进行处理，这些构件包括筛选(filtering)，投射(projecting),分组（grouping）， 限制(limiting)和跳过(skipping)
 
            Mongodb不允许单一的聚合操作占用过多的系统内存，如果mongodb发现某个聚合操作占用了超过20%内存，这个操作就会被直接输出错误
         2）MapReduce
            有些问题过于复杂，无法使用聚合框架的查询语言来表达，这时可以使用MapReduce，MapReduce使用js作为查询语言，js作为查询语言，因此它能够表达任意复杂的逻辑，然而MapReduce非常慢，不应该用在实时的数据分析中。
 
            MapReduce能够在多台服务器之间并行执行，它会将一个大问题拆分为多个小问题，将多个小问题发送到不同机器上，每台机器只负责完成一部分工作，所有机器都完成后，再将这些零碎的解决方案合并为一个完整的解决方案
           
            MapReduce的几个步骤
              1）映射（map）
	             将操作映射到集合中的每一个文档
	          2）中间环节 洗牌 (shuffle)
	             按照键分组，将产生的键值组成列表放到对应的键中
	          3）化简(reduce)
                 把列表中的值化简称一个单值，这个值被返回，然后接着进行洗牌，直到每个键的列表只有一个值为止，这个值就是最终结果
         3) 聚合命令
            1) count
            2) distinct
            3) group 
7）复制
     1）创建副本集
     2）副本集的心跳，选举，回滚
     3）从应用程序连接到副本集
     5）副本集的管理
     6）分片
        1）分片概念
        2）配置分片
        3）追踪集群数据
     7）选择片键
     8）分片管理
     9）数据管理
        1）建立删除索引
        2）预热数据
        3）压缩数据
        5）移动集合
     10）持久性
     11）Mongodb监控
     12) 备份
        1) 文件系统快照
        2) 复制数据文件
        3) 对副本集进行备份
        5) 对分片集群进行备份
     13) Mongodb环境部署
     15) 应用程序设计
         范式化：
            将数据分散到不同的集合，不同集合之间可以相互引用数据，但是引用的数据只存储在一个集合中
         反范式化：	
            将每个文档所需的数据都嵌入在文档内部，每个文档都拥有自己的数据副本，而不是所有文档共同引用同一个数据副本
</pre>

<pre>
SequoiaDB技术
</pre>
