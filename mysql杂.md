[TOC]

# 事务

**幻读**

幻读，并不是说两次读取获取的结果集不同，幻读侧重的方面是某一次的 select 操作得到的结果所表征的数据状态无法支撑后续的业务操作。更为具体一些：select 某记录是否存在，不存在，准备插入此记录，但执行 insert 时发现此记录已存在，无法插入，此时就发生了幻读。

**发生过程**

**非unique字段**。当想只插入一个name=chi的记录时，先select判断是否存在，然后再insert，会有几率发生幻读。

| 事务A | B/事务B |                    |
| :------- | :--------- | ------ |
| begin; | /begin;   | |
|  | Insert into t(name) values('chi'); |  |
| select count(1) from t where name='chi'; 结果为0。发生幻读  |  |幻读|
| 判断如果count(1)为0，执行Insert into t(name) values('chi'); |              ||
| commit;                                                     | |数据库中出现2个记录name=’chi'|

解决方案:

1. [有些情况下不一定，因为select 没加锁都会存在幻读，要都加]mysql的默认事务隔离级别repeatable-read，对SELECT 操作也手动加锁。select count(1) from t where name='chi' for update; 
   1. 已经有其他事务在操作t，此事务会被挂起，直到其他都提交，并且select的数据是最新的，如果之前有另一个没有锁的select，这两个select结果可能会不一样。
   2. 没有其他事务，会拿到锁，当有其他事务操作t时，会被挂起。
2. 改变事务隔离界别成SERIALIZABLE。

**unique字段**不会发生幻读，是因为InnoDB的行锁锁定了索引

| 事务A | B/事务B                      |                    |
| :---- | :-------------- | ---------------------- |
| begin; | /begin;  | |
| Insert into t(id) values(1); |             |  |
|   | Insert into t(id) values(1); |mysql检测到unique字段相同，无论后插入的B是否为事务，都会被挂起，直到事务A提交或者超时报错。|
| commit； |              |事务B|