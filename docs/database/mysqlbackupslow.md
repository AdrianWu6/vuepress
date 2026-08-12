---
title: 分析MysqlDump恢复慢点原因
date: 2026-07-29
categories:
  - DataBase
tags:
  - MySql
---
## 问题描述
主服务器备份脚本
`--single-transaction --quick`这里是必要的，备份的时候会根据当前时点快照备份，如果不加，全库备份的时候只能query不能execute，会影响业务。

```shell
TODAY=`date +%Y%m%d`
mysqldump -ueds -peds111 --single-transaction --quick eds|gzip > date_${TODAY}.sql.gz
ftp
....
EOF
find -name "data*.sql.gz" -print -mtime +1 -exec rm {} \;
```
备份数据库恢复脚本
```shell
TODAY=`date +%Y%m%d`
let NUM=1
while true
do
	if [ -f data_${TODAY}.sql.gz ]
	then 
		gzip -dc data_${TODAY}.sql.gz|mysql -ueds -peds111 eds
		break
	else
		sleep 600
		continue
	fi
	let NUM++
	if [ ${NUM} -gt 36 ]
	then		
		break
	fi
done
	
```
随着数据库数据的连年增长，data_${TODAY}.sql.gz的大小达到了20G，恢复一次需要花两天时间，需要排查原因对其优化。
## 排查过程
### CPU与内存
```
top
%CPU 16 %MEM 4.4
```
CPU占用和内存占用都很低，不是瓶颈所在
```
vmstat 1
wa 之在90%以上
```
### 重要配置查询
```
show variables like 'log_bin';   OFF
show variables like 'innodb_flush_log_at_trx_commit' 1
```
可以看出binlog日志处于关闭状态，瓶颈不在此处，日志模式打开了，数据比较多，可以作为怀疑点之一。

日志模式关闭后
速度提高了23%
### 系统基础排查

