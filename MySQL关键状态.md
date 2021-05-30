# 关键参数说明

[参考资料](https://www.cnblogs.com/AloneSword/p/3207697.html)   
[参考资料](https://developer.aliyun.com/article/516665)   

![](images/3/07.jpg)   

# show global status

```

Aborted_clients
客户端退出之前，没有调用mysql_close()
客户端“沉睡”，超过wait_timeout 或者 interactive_timeout设置时间（单位：秒）没有向服务器发起任何请求。
客户端程序在数据传输的期间突然终止。

Aborted_connects
客户端因为没有权限访问数据库导致的连接失败
客户端因为密码错误导致的连接失败

临时表 tmp_table
created_tmp_tables 实例运行开始到现在创建临时表数量,内存中
created_tmp_disk_tables 在磁盘上创建临时表,内部内存临时表的最大大小,实际限制由tmp_table_size和max_heap_table_size中的较小值确定,默认16M,超过此参数设置就要创建磁盘临时表
created_tmp_files  表示mysql服务创建的临时文件文件数

表扫描情况
https://blog.csdn.net/n88Lpo/article/details/102713594
Handler_read_first
此选项表明SQL是在做一个全索引扫描，注意是全部，而不是部分，所以说如果存在WHERE语句，这个选项是不会变的
如果这个选项的数值很大，既是好事也是坏事。说它好是因为毕竟查询是在索引里完成的，而不是数据文件里，
说它坏是因为大数据量时，即使是索引文件，做一次完整的扫描也是很费时间的。
Handler_read_key  不管全表全索引或者正确使用的索引实际上都会增加，只是一次索引定位而已。
Handler_read_last 取索引中最后一个键的请求数,使用ORDER BY desc
Handler_read_next 
一个累计值:
主键索引的范围查询
唯一索引的范围查询
非唯一索引的等值查询
非唯一索引的范围查询

全表扫描:Handler_read_first增加1次用于初次定位，Handler_read_key增加1次，Handler_read_rnd_next增加扫描行数
全索引扫描:Handler_read_first增加1次用于初次定位，Handler_read_key增加1次，Handler_read_next增加扫描行数用于连续访问接下来的行
索引ref访问:Handler_read_key增加1次这是用于初次定位，Handler_read_next增加扫描行数次数用于接下来的数据访问。
索引range访问:Handler_read_key增加1次这是用于初次定位，Handler_read_next增加扫描行数次数用于接下来的数据访问。
索引反向排序:Handler_read_last增加1次这是用于初次定位, Handler_read_prev增加扫描行数次数用于接下来的数据访问。
详细参考:https://mytecdb.com/blogDetail.php?id=43


Table_locks_immediate，产生表级锁的次数。
Table_locks_waited，数显表级锁而等待的次数
行级锁可以通过下面几个变量查询：
Innodb_row_lock_current_waits，当前正在等待锁定的数量。
Innodb_row_lock_time(重要)，从系统启动到现在锁定总时长。
Innodb_row_lock_time_avg(重要)，每次等待所花平均时间。
Innodb_row_lock_time_max，从系统启动到现在等待最长的一次花费时间。
Innodb_row_lock_waits(重要)，从系统启动到现在总共等待的次数。
Innodb_buffer_pool_wait_free:
等待innodb刷新脏页到磁盘的计数
如果innodb_buffer_pool_size设置够大，那么此值应很小，
如果不为0且在持续增加，说明当前innodb_buffer_pool_size严重不足，需要加大

Innodb_log_waits
innodb redo log写发生的等待次数，可能因为日志缓冲区太小，
导致写redo log buffer时需要等待，如果这个大于0，就表示innodb_log_buffer不够用了，需要加大

pen_tables 当前打开的表的数量
Opened_files 已经打开的表的数量。如果Opened_tables较大，table_cache 值可能太小

Threads_connected
当前打开的连接的数量。包含内部线程和sleep状态的线程
hreads_created
表示创建过的线程数，如果发现threads_created值过大的话，表明mysql服务器一直在创建线程
这也是比较耗资源，可以适当增加配置文件中thread_cache_size值，
hreads_running
激活的（非睡眠状态）线程数。也代表MySQL并发用户活动会话数量,在系统负载很大时,SQL持续时间会增加,这个指标会上升

Select_full_join
由于不使用索引而执行表扫描的联接数。如果该值不是0，则应仔细检查表的索引
注意，该缓冲区是每个连接独占的，所以总缓冲区大小为 join_buffer_size*连接数；极端情况1M*maxconnectiosns，会超级大。所以要考虑日常平均连接数。
Select_full_range_join
join查询中被驱动表使用索引范围扫描的select查询数量
Select_scan单表查询或者join的第一个表使用全表扫描方式的select查询数量
Select_range单表查询或者join的第一个表使用索引范围扫描方式的select查询数量

Sort_merge_passes 
表示当需要排序时，在排序缓冲中无法将结果完全存放，则将会基于磁盘创建临时文件进行排序。如果该值较高，
则应提高sort_buffer_size大小。最好的办法是找到是由哪些排序SQL造成的。

```

# show processlist  

[参考资料](https://dev.mysql.com/doc/refman/8.0/en/general-thread-states.html)

