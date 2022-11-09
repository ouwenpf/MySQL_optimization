
## InnoDB行锁的实现机制

- 基于索引实现
- 逐行检查，逐行加锁
- 没有索引的列上需要加锁时，会先对所有记录加锁，再根据实际情况决定是否释放锁(<=RC隔离级别下，虽然对所有记录都加上锁了，但是在执行update的时候会判断where条件和加锁的范围是否冲突，如果不冲突可以执行成功)
- 辅助索引上加锁时，同时要回溯到主键索引上再次加一次锁


## InnoDB锁类型

- InnoDB的几个lock_modes
	- LOCK_IS,表级锁，意向共享锁，表示将要在表上加上共享锁
	- LOCK_IX,表级锁，意向排他锁，表示将要在表上加上排他锁
	- LOCK_S,共享锁，可用于表锁，行锁
	- LOCK_X,排他锁，可用于表锁，行锁
	- LOCK_AUTO_INC，表级auto-inc锁(在SQL执行完就释放，而不是事务执行完释放)


## InnoDB之inserion intention lock

- 它是个gap lock
- 如果两个不同事务向往同一个gap中写入数据，但写入位置不一样，是无需等待，可以直接写人，因为没有冲突
- gap lock仅仅用于防止往gap上写入新记录(避免幻读)，因此无论是S-gap还是X-gap锁，作用是一样的


## InnoDB锁特点

- 显示锁(explicit-lock)
	- select *  from tb_name where rowed='xxxx' for update|lock in share mode
- 隐式锁(implicit-lock)
	- insert|delete|update
	- 任何辅助索引上锁，或非索引列上锁，都要回溯到主键上，也加锁
	- 和其它session有冲突时候，隐式锁转换成显示锁(s2在操作时候发现和s1有冲突的时候，帮s1加上锁，然后自己放到等待锁队列中)


## 一致性非锁定读

- consistent non-locking read
- 通过MVCC机制，基于当前时间点read view读取
- 默认情况下，不加锁



## 重要总结

```
create table t1 (
	c1 int not null,
	c2 int not null,
	c3 int not null,
	c4 int not null,
	primary key (c1),
	unique key c2 (c2),
	key c3 (c3)
);

+----+----+----+----+
| c1 | c2 | c3 | c4 |
+----+----+----+----+
|  1 |  1 |  1 |  1 |
| 10 | 10 | 10 | 10 |
| 20 | 20 | 20 | 20 |
| 30 | 30 | 30 | 30 |
+----+----+----+----+

insert into t1 values(1,1,1,1),(10,10,10,10),(20,20,20,20),(30,30,30,30);

1. 主键等值条件

select * from t1 where c1=1 for update;
1)在c1=1上 LOCK_X|LOCK_REC_NOT_GAP

2. 主键等值条件，更新索引列
update t1 set c3=2 where c1=1;
1)在c1=1上加LOCK_X|LOCK_REC_NOT_GAP
2)在c3=1上加LOCK_X|LOCK_REC_NOT_GAP--> c3=2上加LOCK_X|LOCK_INSERT_INTENTION --> 更新成功后在c3=2上加LOCK_X|LOCK_REC_NOT_GAP
 
3. 主键等值条件，但数据不存在
select *  from t1 where c1=1;
在下一条记录(c1=10)前面加上LOCK_X|LOCK_GAP
update更新也是如此

4. 主键范围条件
select * from  t1 where c1>=20  for update;
1)在c1=20上加LOCK_X|LOCK_REC_NOT_GAP
2)在c1>20上加LOCK_X|LOCK_ORDINARY包括supremum记录

5. 主键范围更新包含索引列
update t1 set c3=32 where c1>=30;
1)在c1=30上加LOCK_X|LOCK_REC_NOT_GAP
2)在c1>30上加LOCK_X|LOCK_ORDINARY包括supremum记录
3)在c3=30上加LOCK_X|LOCK_REC_NOT_GAP--> c3=32上加LOCK_X|LOCK_INSERT_INTENTION --> 更新成功后在c3=32上加LOCK_X|LOCK_REC_NOT_GAP

6. 主键范围条件
select *  from  t1 where c1<=10  for update;
1)在c1<=10上加LOCK_X|LOCK_ORDINARY包括infimum记录
2)先在c1=10下一条记录(c1=20)上加LOCK_X|LOCK_ORDINARY，在server层再判断c1=20不符合条件(RR级别下不释放锁，RC级别下释放锁)

7. 唯一主键加锁的情况和主键索引类似，只不过需要回溯到主键上重新加锁

8. 普通索引等值条件
select *  from  t1 where c3=1  for update;
1)在c3=1上加LOCK_X|LOCK_ORDINARY
2)在c3=1的下一条记录(c3=10)前面的GAP加上LOCK_X|LOCK_GAP
1)在c1=1上加LOCK_X|LOCK_REC_NOT_GAP


9. 普通索引范围条件
select *  from  t1 where c3<=10  for update;
1)在c3<=10上加LOCK_X|LOCK_ORDINARY包括infimum记录
2)先在c3=10下一条记录(c1=20)上加LOCK_X|LOCK_ORDINARY，在server层再判断c1=20不符合条件(RR级别下不释放锁，RC级别下释放锁)
3)在相应主键c1=1加上LOCK_X|LOCK_REC_NOT_GAP
   在相应主键c1=10加上LOCK_X|LOCK_REC_NOT_GAP

10. 普通索引范围条件

select *  from  t1 where c3>=1  for update;
1)在c3>=1上加LOCK_X|LOCK_ORDINARY包括supremum记录
2)在相应主键c1=1加上LOCK_X|LOCK_REC_NOT_GAP
   在相应主键c1=10加上LOCK_X|LOCK_REC_NOT_GAP
   在相应主键c1=20加上LOCK_X|LOCK_REC_NOT_GAP
   在相应主键c1=30加上LOCK_X|LOCK_REC_NOT_GAP
   
   
11. 无索引加锁会所有主键上LOCK_X|LOCK_ORDINARY，相当于锁定表
```


## InnoDB锁总结

- 默认都是加next-key lock
- 如果是唯一索引(含主键索引)上的等值查询，则退化成LOCK_REC_NOT_GAP
- 非唯一索引上的等值查询，向右遍历遇到最后一个不符合记录时先加上next-key lock，而后退化成LOCK_GAP
- 由于设计/代码上的"缺陷"，唯一索引(含主键索引)上的小于范围查询时，遇到第一个不符合条件的记录也会加上LOCK_ORDINARY(RC级别下先获取再释放，RR级别下不释放)


http://keithlan.github.io/