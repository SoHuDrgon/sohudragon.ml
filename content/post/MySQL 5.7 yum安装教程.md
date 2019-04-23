---
title: "MySQL 5.7 yum安装教程"
date: 2019-04-23T11:16:47+08:00
tags: [“MySQL”,"MySQL 5.7"]
categories: [“MySQL”]
draft: false
---
## 准备工作
### 我们要先下载mysql的repo源。

`wget https://dev.mysql.com/get/mysql57-community-release-el6-11.noarch.rpm`

### 安装repo源：
```
rpm -ivh mysql57-community-release-el6-11.noarch.rpm 
warning: mysql57-community-release-el6-11.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID 5072e1f5: NOKEY
Preparing...                ########################################### [100%]
   1:mysql57-community-relea########################################### [100%]
```
或者直接安装：
`rpm -ivh https://dev.mysql.com/get/mysql57-community-release-el6-11.noarch.rpm`

#### 检测系统是否自带安装mysql
```
yum list installed | grep mysql
mysql57-community-release.noarch    el6-11                              installed
```

### 删除系统自带的mysql及其依赖
命令：

`yum -y remove mysql-libs.x86_64`

上面有依赖，我们用下面的命令：
`rpm -e mysql-libs.x86_64 --nodeps`

## 安装MySQL服务器

### 安装命令：
`yum install mysql-community-server`

### 配置mysql配置文件my.cnf

#### 基础配置
```
mkdir /data
cd /data
mkdir mysql/{data,log,run,binlogs,undolog,tmp} -p
chown -R mysql:mysql /data/mysql/
```


#### 配置文件如下：
```
# cat /etc/my.cnf
# Mysql conf
[client]
port                            = 3306
socket                          = /data/mysql/tmp/mysql.sock
default-character-set           = utf8mb4

[mysql]
no-auto-rehash
default-character-set           = utf8mb4

[mysqld]
user                            = mysql
socket                          = /data/mysql/tmp/mysql.sock
port                            = 3306
#character-set-server           = utf8
#collation-server               = utf8_general_ci
init-connect                    = 'SET NAMES utf8mb4'
character-set-server            = utf8mb4
collation-server                = utf8mb4_general_ci
server-id                       = 600
pid-file                        = /data/mysql/run/mysql.pid
#basedir                         = /usr/local/mysql
datadir                         = /data/mysql/data
master_info_repository          = TABLE
relay_log_info_repository       = TABLE
relay_log_recovery              = 1
relay-log-purge                 = 1
relay-log                       = /data/mysql/binlogs/relay-bin
##半同步复制开启
#rpl_semi_sync_master_enabled   = 1    
#rpl_semi_sync_master_timeout   = 10000 # 10 second·
# LOG
log-bin                         = /data/mysql/binlogs/binlog-mysqld
binlog_format                   = row
log_slave_updates
expire_logs_days                = 7
long_query_time                 = 1
slow-query-log
slow_query_log_file             = /data/mysql/log/slow.log
log-error                       = /data/mysql/log/error.log

##UNDO LOG Configure
innodb_undo_directory           = /data/mysql/undolog
innodb_undo_tablespaces         = 4
innodb_undo_logs                = 128
innodb_max_undo_log_size        = 1G
innodb_purge_rseg_truncate_frequency
innodb_undo_log_truncate        = 1

##INNODB TEMP DATA Configure
#innodb_temp_data_file_path=tmp_space2/ibtmp2:200M:autoextend


gtid_mode                       = on
enforce_gtid_consistency        = 1
sync-master-info                = 1
slave-parallel-workers          = 4
binlog-checksum                 = CRC32
master-verify-checksum          = 1
slave-sql-verify-checksum       = 1
binlog-rows-query-log_events    = 1

default-storage-engine          = INNODB
skip-name-resolve
skip_external_locking
lower_case_table_names          = 1
event_scheduler                 = 0
back_log                        = 512
max_connections                 = 8192
max_connect_errors              = 99999
max_allowed_packet              = 64M
max_heap_table_size             = 256M
max_length_for_sort_data        = 16k
wait_timeout                    = 172800
interactive_timeout             = 172800
net_buffer_length               = 8K
read_buffer_size                = 4M
read_rnd_buffer_size            = 4M
join_buffer_size                = 2M
sort_buffer_size                = 2M
binlog_cache_size               = 2M
table_open_cache                = 2048
table_definition_cache          = 2048
thread_cache_size               = 32
tmp_table_size                  = 128M
long_query_time                 = 1
log_error_verbosity             = 3
slave-skip-errors               = 1022,1032,1062
log_slave_updates               = 1
log_bin_trust_function_creators = 1
explicit_defaults_for_timestamp = 1
#sql_mode='STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION'
sql_mode='NO_ENGINE_SUBSTITUTION,NO_AUTO_CREATE_USER'

max_binlog_size                 = 1G
sync_binlog                     = 0
expire_logs_days                = 7
key_buffer_size                 = 384M
bulk_insert_buffer_size         = 16M
myisam_sort_buffer_size         = 64M
myisam_max_sort_file_size       = 10G
myisam_repair_threads           = 1

innodb_file_per_table
innodb_buffer_pool_size         = 16G
innodb_data_file_path           = ibdata1:2048M:autoextend
innodb_flush_log_at_trx_commit  = 2
innodb_fast_shutdown            = 1
innodb_log_buffer_size          = 8M
innodb_log_file_size            = 1000M
innodb_max_dirty_pages_pct      = 75
innodb_lock_wait_timeout        = 120
innodb_rollback_on_timeout      = 1
innodb_flush_method             = O_DIRECT
innodb_old_blocks_time          = 1000
memlock
innodb_support_xa               = on
transaction_isolation           = REPEATABLE-READ
innodb_buffer_pool_instances    = 10
innodb_read_io_threads          = 4
innodb_write_io_threads         = 10
innodb_use_native_aio           = 1
innodb_file_format              = barracuda
innodb_file_format_check        = ON
innodb_strict_mode              = 1
innodb_purge_threads            = 1
innodb_change_buffering         = inserts
innodb_commit_concurrency       = 64
innodb_thread_concurrency       = 128

# 根据您的服务器IOPS能力适当调整
# # 一般配普通SSD盘的话，可以调整到 10000 - 20000
# # 配置高端PCIe SSD卡的话，则可以调整的更高，比如 50000 - 80000
#innodb_io_capacity             = 50000
#innodb_io_capacity_max                 = 80000

# 忽略mysql系统库复制
binlog-ignore-db                = mysql
binlog-ignore-db                = information_schema
binlog-ignore-db                = performance_schema
binlog-ignore-db                = test
replicate-ignore-db             = test
replicate-ignore-db             = mysql
replicate-ignore-db             = information_schema
replicate-ignore-db             = performance_schema
slave-skip-errors               = 1022,1032,1062

[mysqldump]  
quick  
max_allowed_packet              = 32M  
                                                                                                      
                                                                                                      
[myisamchk]  
key_buffer_size                 = 8M  
sort_buffer_size                = 8M  
                                                                                                      
[mysqlhotcopy]  
interactive-timeout  
                                                                                                                                                                                               
[mysqld_safe]  
log-error                       = /data/mysql/log/error.log  
pid-file                        = /data/mysql/run/mysql.pid 
open-files-limit                = 8192
```

### 启动mysql：
`/etc/init.d/mysqld start`


### 初始化启动后密码存储在日志里面：
```
[root@xxx ~]# grep "temporary password" /data/mysql/log/error.log
2017-08-14T20:22:22.872814Z 1 [Note] A temporary password is generated for root@localhost: &WGIA<CyE4I3
```

```
[root@xxx ~]# mysql -uroot -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 5
Server version: 5.7.19-log

Copyright (c) 2000, 2017, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
提示必须重置密码：
mysql> show databases;
ERROR 1820 (HY000): You must reset your password using ALTER USER statement before executing this statement.
密码必须符合要求：
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY '2544gxrr';
ERROR 1819 (HY000): Your password does not satisfy the current policy requirements

mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'MyNewPass4!';
Query OK, 0 rows affected (0.00 sec)
或者：
mysql> set password for 'root'@'localhost'=password('MyNewPass4!'); 
```

## 查看和更改mysql密码策略
### 使用如下命令查看现有的密码策略
`SHOW VARIABLES LIKE 'validate_password%';`

详细说明：
**alidate_password_number_count**参数是密码中至少含有的数字个数，当密码策略是MEDIUM或以上时生效。  

**validate_password_special_char_count**参数是密码中非英文数字等特殊字符的个数，当密码策略是MEDIUM或以上时生效。  

**validate_password_mixed_case_count**参数是密码中英文字符大小写的个数，当密码策略是MEDIUM或以上时生效。  

**validate_password_length**参数是密码的长度，这个参数由下面的公式生成  

validate_password_number_count+ validate_password_special_char_count+ (2 * validate_password_mixed_case_count)  

**validate_password_dictionary_file**参数是指定密码验证的字典文件路径。  

**validate_password_policy**这个参数可以设为0、1、2，分别代表从低到高的密码强度，此参数的默认值为1，如果想将密码强度改弱，则更改此参数为0。  

### 最后附上mysql的密码策略更改语句：

更改密码策略为LOW     

`set global validate_password_policy=0;`

更改密码长度  
`set global validate_password_length=0;`





