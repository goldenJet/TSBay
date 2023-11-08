## SQL 相关操作

#### 连接

``` 
mysql -h 192.1.1.1 -u root -p
```


#### group_concat

group_concat 指定排序规则 & 指定连接的字符：
`GROUP_CONCAT(name order by id asc SEPARATOR '/')`


#### 时间

DATE_FORMAT() 主要参数：

|字段 |描述 |demo |
|:---- |:---- |:---- |
|%Y |年 |2020 |
|%m |月 |04 |
|%d |日 |07 |
|%T |时间 24-小时 (`hh:mm:ss`) |`09:03:03` |
|%H |时 |09、21|
|%i |分 |03 |
|%s |秒 |03 |
|%w |星期 |0=星期日, 6=星期六 |

#### 查询结果添加一个排序列

比如查询的结果想要增加一列，内容为：3、4、5、6。。。
```sql
set @rn=2;
SELECT
    (@rn:=@rn+1) as id,
    ...
```

## 数据库死锁解决办法
step1：查看当前事务： `select * from information_schema.INNODB_TRX;`
step2：杀掉抢资源的线程：`Kill [trx_mysql_thread_id]；`

> 另：  
> 查看正在锁的事务: SELECT * FROM INFORMATION_SCHEMA.INNODB_LOCKS;  
> 查看等待锁的事务: SELECT * FROM INFORMATION_SCHEMA.INNODB_LOCK_WAITS;

查询谁阻塞和谁在等待，以及等待了多久：
```sql
SELECT
    r.trx_id waiting_trx_id,
    r.trx_mysql_thread_id waiting_thread,
    TIMESTAMPDIFF( SECOND, r.trx_wait_started, CURRENT_TIMESTAMP ) wait_time,
    r.trx_query waiting_query,
    l.lock_table waiting_table_lock,
    b.trx_id blocking_trx_id,
    b.trx_mysql_thread_id blocking_thread,
    SUBSTRING( p.HOST, 1, INSTR( p.HOST, ':' ) - 1 ) blocking_host,
    SUBSTRING( p.HOST, INSTR( p.HOST, ':' ) + 1 ) blocking_port,
IF
    ( p.COMMAND = 'Sleep', p.TIME, 0 ) idel_in_trx,
    b.trx_query blocking_query 
FROM
    information_schema.INNODB_LOCK_WAITS w
    INNER JOIN information_schema.INNODB_TRX b ON b.trx_id = w.blocking_trx_id
    INNER JOIN information_schema.INNODB_TRX r ON r.trx_id = w.requesting_trx_id
    INNER JOIN information_schema.INNODB_LOCKS l ON w.requested_lock_id = l.lock_id
    LEFT JOIN information_schema.PROCESSLIST p ON p.ID = b.trx_mysql_thread_id 
ORDER BY
    wait_time DESC
```

查询有多少查询被哪些线程阻塞
```sql
SELECT
    CONCAT( 'thread ', b.trx_mysql_thread_id, ' from ', p.HOST ) AS who_blocks,
IF
    ( p.command = "Sleep", p.time, 0 ) AS idle_in_trx,
    MAX( TIMESTAMPDIFF( SECOND, r.trx_wait_started, NOW( ) ) ) AS max_wait_time,
    COUNT( * ) AS num_waiters 
FROM
    INFORMATION_SCHEMA.INNODB_LOCK_WAITS AS w
    INNER JOIN INFORMATION_SCHEMA.INNODB_TRX AS b ON b.trx_id = w.blocking_trx_id
    INNER JOIN INFORMATION_SCHEMA.INNODB_TRX AS r ON b.trx_id = w.requesting_trx_id
    LEFT JOIN INFORMATION_SCHEMA.PROCESSLIST AS p ON p.id = b.trx_mysql_thread_id 
GROUP BY
    who_blocks 
ORDER BY
    num_waiters DESC
```

## 查看状态
show status;  -- mysql 数据库状态

查状态语法：show status like '%变量名%'

如：
``` mysql
>show status like 'Threads%'; -- 连接线程数  
SHOW STATUS LIKE '%Connection%';    
show global status like 'Max_used_connections'; -- 已使用连接数  
show variables like '%max_connections%';  -- 最大连接数  
SHOW FULL PROCESSLIST;  -- 连接详细信息  
```

```
-- 设置最大连接数
-- SET GLOBAL max_connections=2000;
```

常用变量有：
>Aborted_clients 由于客户没有正确关闭连接已经死掉，已经放弃的连接数量。   
Aborted_connects 尝试已经失败的MySQL服务器的连接的次数。  
Connections 试图连接MySQL服务器的次数。  
Created_tmp_tables 当执行语句时，已经被创造了的隐含临时表的数量。  
Delayed_insert_threads 正在使用的延迟插入处理器线程的数量。  
Delayed_writes 用INSERT DELAYED写入的行数。  
Delayed_errors 用INSERT DELAYED写入的发生某些错误(可能重复键值)的行数。  
Flush_commands 执行FLUSH命令的次数。  
Handler_delete 请求从一张表中删除行的次数。  
Handler_read_first 请求读入表中第一行的次数。  
Handler_read_key 请求数字基于键读行。  
Handler_read_next 请求读入基于一个键的一行的次数。  
Handler_read_rnd 请求读入基于一个固定位置的一行的次数。  
Handler_update 请求更新表中一行的次数。  
Handler_write 请求向表中插入一行的次数。  
Key_blocks_used 用于关键字缓存的块的数量。  
Key_read_requests 请求从缓存读入一个键值的次数。  
Key_reads 从磁盘物理读入一个键值的次数。  
Key_write_requests 请求将一个关键字块写入缓存次数。  
Key_writes 将一个键值块物理写入磁盘的次数。  
Max_used_connections 同时使用的连接的最大数目。  
Not_flushed_key_blocks 在键缓存中已经改变但是还没被清空到磁盘上的键块。  
Not_flushed_delayed_rows 在INSERT DELAY队列中等待写入的行的数量。  
Open_tables 打开表的数量。  
Open_files 打开文件的数量。  
Open_streams 打开流的数量(主要用于日志记载）  
Opened_tables 已经打开的表的数量。  
Questions 发往服务器的查询的数量。  
Slow_queries 要花超过long_query_time时间的查询数量。  
Threads_connected 当前打开的连接的数量。  
Threads_running 不在睡眠的线程数量。  
Uptime 服务器工作了多少秒。  


## 时间函数

- 今天

``` sql
SELECT DATE_FORMAT(NOW(),'%Y-%m-%d 00:00:00') AS '今天开始';
SELECT DATE_FORMAT(NOW(),'%Y-%m-%d 23:59:59') AS '今天结束';
```

- 昨天
``` sql
SELECT DATE_FORMAT( DATE_SUB(CURDATE(), INTERVAL 1 DAY), '%Y-%m-%d 00:00:00') AS '昨天开始';
SELECT DATE_FORMAT( DATE_SUB(CURDATE(), INTERVAL 1 DAY), '%Y-%m-%d 23:59:59') AS '昨天结束';
```

- 上周
``` sql
SELECT DATE_FORMAT( DATE_SUB( DATE_SUB(CURDATE(), INTERVAL WEEKDAY(CURDATE()) DAY), INTERVAL 1 WEEK), '%Y-%m-%d 00:00:00') AS '上周一';
SELECT DATE_FORMAT( SUBDATE(CURDATE(), WEEKDAY(CURDATE()) + 1), '%Y-%m-%d 23:59:59') AS '上周末';
```

- 本周
``` sql
SELECT DATE_FORMAT( SUBDATE(CURDATE(),DATE_FORMAT(CURDATE(),'%w')-1), '%Y-%m-%d 00:00:00') AS '本周一';
SELECT DATE_FORMAT( SUBDATE(CURDATE(),DATE_FORMAT(CURDATE(),'%w')-7), '%Y-%m-%d 23:59:59') AS '本周末';
```

- 上面的本周算法会有问题,因为mysql是按照周日为一周第一天,如果当前是周日的话,会把时间定为到下一周.
``` sql
SELECT DATE_FORMAT( DATE_SUB(CURDATE(), INTERVAL WEEKDAY(CURDATE()) DAY), '%Y-%m-%d 00:00:00') AS '本周一';
SELECT DATE_FORMAT( DATE_ADD(SUBDATE(CURDATE(), WEEKDAY(CURDATE())), INTERVAL 6 DAY), '%Y-%m-%d 23:59:59') AS '本周末';
```

- 上月
``` sql
SELECT DATE_FORMAT( DATE_SUB(CURDATE(), INTERVAL 1 MONTH), '%Y-%m-01 00:00:00') AS '上月初';
SELECT DATE_FORMAT( LAST_DAY(DATE_SUB(CURDATE(), INTERVAL 1 MONTH)), '%Y-%m-%d 23:59:59') AS '上月末';
```

- 本月
``` sql
SELECT DATE_FORMAT( CURDATE(), '%Y-%m-01 00:00:00') AS '本月初';
SELECT DATE_FORMAT( LAST_DAY(CURDATE()), '%Y-%m-%d 23:59:59') AS '本月末';
```


