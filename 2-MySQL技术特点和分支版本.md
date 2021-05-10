# MySQL技术特点

[Release Notes](https://dev.mysql.com/doc/relnotes/mysql/8.0/en/)  
[my.cnf生成器](https://zhishutang.com/my-cnf-wizard.html)   
[my.cnf生成器](https://imysql.com/my-cnf-wizard.html)
## 1.理解MySQL特点
- 单进程,多线程
- 每个连接只能用到一个逻辑CPU
- 每个Query/SQL只能用到一个逻辑CPU
<pre>
并行读目前只有一种情况,就是在执行check table和select count(*)的时候
innodb表支持在聚集索引上的并行读,即对主键索引的并行读,其它目前都不支持
具体说明请看https://www.cnblogs.com/cchust/p/12347166.html
</pre>
- 在超高并发且多垃圾SQL的情况下,对MySQL而言是个大灾难
- 不使用MySQL 5.1以前的版本,因为多核CPU利用更差
- MySQL新版本高并发可以很好利用多核CPU
- 使用/优化建议
	- 使用新版本,抛弃旧版本
	- 用高主频,多核CPU
	- 不跑复杂的SQL
	- 事务及早结束
	
## 2.理解MySQL特点-内存
- MySQL 5.7开始可以在线动态调整IBP,也不建议设置过高或者过低
```
innodb_buffer_pool_chunk_size定义buffer_pool块的大小默认128M
innodb_buffer_pool_size为innodb_buffer_pool_chunk_size为整数倍,默认是整数倍的时候可以动态调整
至于原因可以查询图解参数说明-01.jpg

```
- 每个session buffer诸如sort/join/read buffer/tmp table不宜设置太高容易OMM
- 使用/优化建议
	- IBP一般最高设置物理内存的50%-70%(实际上有多10%的消耗,额外的内存不算在buffer_pool)
	- 加大物理内存,较少物理I/O,提高TPS
	- session级buffer按需分配,因此适当就好,无需过大
	- 禁用所谓的高速查询缓存(query cache),是影影响高并发性能的鸡肋
	- 随着MyISAM引擎逐步被淘汰,key buffer只需要设置非常小
		``` key_buffer_size设置8M-16M```


## 3.理解MySQL特点-I/O-网络

- 磁盘I/O是数据库应用场景最大的瓶颈
- OLTP业务场景中,绝大多数是随机I/O读写 
- 网络一般不会 出现瓶颈,如果是MGR建议换成万兆 网卡
- 使用/优化建议
	- 加大物理内存,减少物理I/O
	- 采用高速磁盘设备设备
	- 适当创建索引,减少随机读
	


## 4.理解其它MySQL特点

- 不存图片,文件,长文本等大数据对象
- 不跑复杂SQL,表达式运算,函数运算等(底层是采用虚拟列,会有 额外的开销)
- 不跑长事务(长时间不提交,不回滚的事务) ,大事务(同一个事务内大量数据修改,插入,更新)
- 不跑全文检索
- 不支持bitmap索引

注意:8.0支持hash join,直方图,倒序索引,不可见索引,跳跃式索引扫描(index skip scan)


## 5.理解其它MySQL特点

- 开源,免费,跨平台
- 和linux深度结合
- 特别适合互联网应用场景 
- 入门学习成本低
- 社区庞大,生态完善
- 人才储备充足
- 新版本新功能越来越赞,值得信赖
- 总之可以放心使用
	



## 本节小结
- 用更好的CPU,更多内存,更好的I/O设备跑MySQL
- 不存大对象数据
- 不跑大事务,垃圾sql,长事务
- 不做复杂的sql运算



# MySQL分支版本

## mariaDB
- 创始人Monty另起炉灶的新分支
- 主要是在server层做了一些改进,增强,以hash join著称
- 最早引入线程池,审计,PAM认证,GTID,InnoDB加密,虚拟列,JSON类型,多远复制,并行复制,CTE,窗口函数,CHECH约束,DEFFAULT之表达式等新功能
- 基础ColumaStore,MyRocks,TokuDB,Spider等多种引擎,插件
- 集成了MaxScale中间件解决方案
- 最新的Oracle MySQL 8.0发布后,metadate全部采用Innodb引擎,MariaDB暂时无法兼容   
[官网](https://www.mariadb.com)   
[详情](https://mariadb.com/kb/en/incompatibilities-and-feature-differences-between-mariadb-102-and-mysql-57/)



## Percona
- Percona Server For MySQL基于官方版本附加性能提升以及管理增强,也集成了TokuDB,线程池,审计,PAM认证等功能
- Percona XtraDB Cluster(PXC)基于gelera高一致性架构
- Percona TokuDB高性能压缩存储引擎
- Xtrabackup备份工具
- Percona-toolkit,DBA附加工具
- PMM监控工具  
[官网](https://www.percona.com)


## MySQL vs MariaDB vs Percona
- MySQL社区版基于GPLv2,企业版则是商用的,而MariaDB只能基于GPLv2,MaxScale商业,ColumnStore基于GPLv2
- MariaDB更激进,更乐于接受社区提交的代码,Oracle MySQL相对保守,几乎很少有直接采用社区贡献的代码(腾讯快速加列功能除外)
- MariaDB和Oracle MySQL的GTID模式不兼容
- Percona则几乎是紧跟追随Oracle MySQL节凑

## 本节小结
- 三个主流分支Oracle MySQL,Percona MySQL,MariaDB
- 优先顺序从左到右
- 选择Percona还是MariaDB需要认真权衡利弊

