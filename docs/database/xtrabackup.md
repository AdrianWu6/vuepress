---
title: xtrabackup MySQL的物理备份
date: 2026-08-07
categories:
  - DataBase
tags:
  - MySql
---
## 什么是xtrabackup
Percona XtraBackup is a 100% open source backup solution for all versions of Percona Server for MySQL and MySQL® that performs online non-blocking, tightly compressed, highly secure full backups on transactional systems.
## 为什么要用xtrabackup
Mysql原生的MysqlDump是逻辑备份，本质上是导出ddl、dml文件。这样做的好处是可移植性强，不受版本约束。但是坏处是导入速度非常的慢，在我们生产环境中生产上的的MysqlDump文件大小高达20G，恢复的时候大概要48h。
XtraBackup是目前全球范围内的唯一一个开源的支持Mysql物理备份软件，他的好处毋庸置疑就是速度很快，直接将Mysql的Data进行备份，但是坏处就是可移植性一般而且要求两端服务器Mysql版本一致，和配置一致。

## 授权Mysql用户权限

* 用Mysql的root用户登入

`mysql -uroot -proot`

* 看下当前数据库中的user和host
`SELECT user,host FROM mysql.user;`
* 授权备份用户权限
`GRANT BACKUP_ADMIN,PROCESS,RELOAD,LOCK TABLES,REPLICATION CLIENT,SELECT ON *.* TO 'user'@'host';`
* 刷新权限
`FLUSH PRIVILEGES;`

## 准备工作
由于环境千奇百怪，直接使用xtrabackup的rpm包总会有问题，直接使用generic二进制版的程序更加稳定。
Liunx root用户登录，把xtrabackup放到/opt/backup下
## 压缩文件
xtrabackup.sh
```shell

#!/bin/bash

  

#常量

DATE=$(date +%Y%m%d)

#备份目录

backupFile=/back/full_${DATE}

#日志

log_file=/backup/logs/${DATE}.log

#数据库配置

dbUsername=edspure

dbPassword='eds@pure@123'

#FTP配置

ftpip=10.200.201.43

username=root

password='eds@pure@123'

#xtrback路径

xtraback=/opt/xtrabackup/bin

  

mkdir -p ${backupFile}

mkdir -p /backup/logs

  

#四个线程压缩备份

echo "=======================backup start ${date +%Y%m%d_%H:%M%S}=========================="

${xtraback}/xtrabackup --backup --user=${dbUsername} --password=${dbPassword} --target-dir=${backupFile} --parallel=4 --compress -compress-threads=4 >>${log_file} 2>&1

  

if [ $? -ne 0 ];then

echo "===================backup failed $(date +%Y%m%d_%H:%M:%S)=================" >> ${log_file}

exit 1

fi

echo "===================backup success $(date +%Y%m%d_%H:%M:%S)=================" >> ${log_file}

  

cd /backup

  

#打个tar包方便ftp传输，减少小文件，提高效率

tar cf full_${DATE}.tar full_${DATE}

  

echo "===================ftp start $(date +%Y%m%d_%H:%M:%S)================="

touch full_${DATE}.ok

ftp -inv ${ftpip} << EOF

user ${username} ${password}

cd /backup

bin

prompt off

put full_${DATE}.tar

put full_${DATE}.ok

EOF

  

if [ $? -ne 0 ];then

echo "===================ftp failed $(date +%Y%m%d_%H:%M:%S)=================" >> ${log_file}

exit 1

fi

echo "===================ftp success $(date +%Y%m%d_%H:%M:%S)=================" >> ${log_file}

  
  

echo "===================remove backdata start $(date +%Y%m%d_%H:%M:%S)=================" >> ${log_file}

if [ -f "full_${DATE}.tar" ] && [ -f "full_${DATE}.ok" ] && [ -d full_${DATE} ]

then

rm -f full_${DATE}.tar full_${DATE}.ok

rm -rf full_${DATE}

echo "===================remove backdata success $(date +%Y%m%d_%H:%M:%S)=================" >> ${log_file}

else echo "===================remove backdata failed $(date +%Y%m%d_%H:%M:%S)=================" >> ${log_file}

fi

  

```
## 恢复数据
xtraReloadData.sh

```shell

#!/bin/bash

  

#常量

DATE=$(date +%Y%m%d)

NUM=1

#备份目录

backupFile=/back/full_${DATE}

#日志

log_file=/backup/logs/${DATE}.log

#数据库配置

dbRootUsername=root

dbRootPassword='root'

#xtrabck路径

xtraback=/opt/xtrabackup/bin

#Mysql路径

mysqlData=/home/mysql/mysql-8.0/

mysqlBin=/usr/local/mysql8.0/bin

  

mkdir -p /backup/logs/

cd /backup

  

echo "startTime is $(date +%Y%m%d_%H:%M:%S)" >> ${log_file}

while true

do

if [ -f full_${DATE}.ok ]

then

#恢复文件解压

tar xf full_${DATE}.tar

cd ${backupFile}

echo "=========== decompress start $(date +%Y%m%d_%H:%M:%S) =============" >> ${log_file}

${xtraback}/xtrabackup --decompress --remove-original -- parallel=4 --target-dir=${backupFile} >> ${log_file} 2>&1

if [ $? -ne 0 ];then

echo "===================decompress failed $(date +%Y%m%d_%H:%M:%S)=================" >> ${log_file}

exit 1

fi

echo "===================decompress success $(date +%Y%m%d_%H:%M:%S)=================" >> ${log_file}

  

#prepare数据

echo "=========== prepare start $(date +%Y%m%d_%H:%M:%S) =============" >> ${log_file}

${xtraback}/xtrabackup --prepare --target-dir=${backupFile} >> ${log_file} 2>&1

if [ $? -ne 0 ];then

echo "===================prepare failed $(date +%Y%m%d_%H:%M:%S)=================" >> ${log_file}

exit 1

fi

echo "===================prepare success $(date +%Y%m%d_%H:%M:%S)=================" >> ${log_file}

  

#停止mysql

${mysqlBin}/mysqladmin -u${dbRootUsername} -p${dbRootPassword} shutdown >> ${log_file} 2>&1

  

#删除老库 创建新DATA

cd ${mysqlData}

  

if [ -d data ]

then

rm -rf data

echo "===================remove data success $(date +%Y%m%d_%H:%M:%S)=================" >> ${log_file}

else

echo "===================remove data failed $(date +%Y%m%d_%H:%M:%S)=================" >> ${log_file}

fi

  

mkdir data

  

#移库

echo "=========== copy-back start $(date +%Y%m%d_%H:%M:%S) =============" >> ${log_file}

  

${xtraback}/xtrabackup --copy-back --target-dir=${backupFile} >> ${log_file} 2>&1

  

if [ $? -ne 0 ];then

echo "===================copy-back failed $(date +%Y%m%d_%H:%M:%S)=================" >> ${log_file}

exit 1

fi

echo "===================copy-back success $(date +%Y%m%d_%H:%M:%S)=================" >> ${log_file}

  

#赋权

chown -R mysql:mysql data

#重启数据库

${mysqlBin}/mysqld --defaults-file=/etc/my.cnf --daemonize >> ${log_file} 2>&1

#删除备份文件

cd /backup

if [ -f "full_${DATE}.tar" ] && [ -f "full_${DATE}.ok" ] && [ -d full_${DATE} ]

then

rm -f full_${DATE}.tar full_${DATE}.ok

rm -rf full_${DATE}

echo "===================remove backdata success $(date +%Y%m%d_%H:%M:%S)=================" >> ${log_file}

else echo "===================remove backdata failed $(date +%Y%m%d_%H:%M:%S)=================" >> ${log_file}

fi

  

break

else

echo "file not exists need wait ten minute" >> ${log_file}

sleep 600

if [ $[NUM] -gt 36 ]

then

echo "wait time too long to break" >>${log_file}

break

fi

fi

done

echo "endtime is $(date +%Y%m%d_%H:%M:%S)"

  

```
## 注意事项
### 版本问题
xtraback选择版本很重要，需要注意的有linux发行版版本、CPU架构、系统位数，然后再选择对应的Mysql版本
`查看发行版 cat /etc/os-release`
`CPU架构 uname -m`
`系统位数 getconf LONG_BIT`
### 两边的Mysql配置要一致
比如我在做备份的时候主库有lower_case_table_names=1，备库没有恢复时就会报错
### 查看Mysql配置
需要确定Mysql的Data和Bin目录位置
`cat /etc/my.cnf`ß