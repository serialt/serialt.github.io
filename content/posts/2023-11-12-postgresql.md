+++
title = 'postgresql'
date = 2023-11-12T10:30:27+08:00
draft = false

tags = ["pg","postgresql","db"]
categories = ["Database"]

+++



# PostgreSQL

## 一、安装

docker 安装

```dockerfile
# docker-compose.yaml
version: "3"
services:
  postgres10-5432:
    image: "postgres:10-bullseye"
    container_name: postgres10-5432
    shm_size: "1gb"
    restart: always
    ports:
      - "5432:5432"
    volumes:
      - /data/postgresql:/var/lib/postgresql/data
      - $PWD/init.sql:/docker-entrypoint-initdb.d/init.sql
    environment:
      - POSTGRES_PASSWORD=xxxxxxxxxxx


# init.sql
CREATE USER db_user WITH CREATEDB ENCRYPTED PASSWORD 'xxxxxxxxxx';
alter user db_user superuser;

```



## 二、配置

### 1、修改数据库时区

```
postgres=# show timezone;
 TimeZone 
----------
 Etc/UTC
(1 row)


sed -i "s+timezone = 'Etc/UTC'+timezone = 'Asia/Shanghai" postgresql.conf
sed -i "s+log_timezone = 'Etc/UTC'+log_timezone = 'Asia/Shanghai'" postgresql.conf


postgres=# select pg_reload_conf();
 pg_reload_conf 
----------------
 t
(1 row)


postgres=# show timezone;
   TimeZone    
---------------
 Asia/Shanghai
(1 row)
```



### 2、sql使用

查询数据库大小：

```sql
select datname, pg_size_pretty (pg_database_size(datname)) AS size from pg_database order by size; 
```

查看表大小

```sql
--数据库中单个表的大小（不包含索引）
select pg_size_pretty(pg_relation_size('表名'));

--查出所有表（包含索引）并排序
SELECT table_schema || '.' || table_name AS table_full_name, pg_size_pretty(pg_total_relation_size('"' || table_schema || '"."' || table_name || '"')) AS size
FROM information_schema.tables
ORDER BY
pg_total_relation_size('"' || table_schema || '"."' || table_name || '"') DESC limit 20;

--查出表大小按大小排序并分离data与index
SELECT
table_name,
pg_size_pretty(table_size) AS table_size,
pg_size_pretty(indexes_size) AS indexes_size,
pg_size_pretty(total_size) AS total_size
FROM (
SELECT
table_name,
pg_table_size(table_name) AS table_size,
pg_indexes_size(table_name) AS indexes_size,
pg_total_relation_size(table_name) AS total_size
FROM (
SELECT ('"' || table_schema || '"."' || table_name || '"') AS table_name
FROM information_schema.tables
) AS all_tables
ORDER BY total_size DESC
) AS pretty_sizes


-- 修改用户密码
ALTER USER postgres with password 'hello_world';

```

schema 管理

```sql
-- 创建schema
create schema test;

-- 查看schema
\dn

-- 修改schema 属主
alter schema test owner to highgo; 

-- 修改schema名称
alter schema test rename to testa; 

create schema test authorization highgo;;

-- 切换schema
set search_path to test_schema
```

修改数据库名

```sql
-- 修改数据库名
alter database src_dbname rename to dst_dbname;

-- 将数据库的名称由database2改成database1
UPDATE pg_database SET datname = 'database1' WHERE datname = 'database2';
```

慢查询配置

```shell
# postgresql.conf
# 10s
log_min_duration_statement=10000

# 热加载配置
postgres=# select pg_reload_conf();
1

# 查看配置：
postgres=# show log_min_duration_statement;

log_min_duration_statement
----------------------------
10s
(1 row)

# 也可以针对某个用户或者某数据库进行设置：
postgres=# alter database test set log_min_duration_statement=5000;


# sql 查询慢语句，超过1s
postgres=# select * from pg_stat_activity where state<>'idle' and now()-query_start > interval '1 s' order by query_start;



select * from pg_stat_activity where state<>'idle' and now()-query_start > interval '5 s' order by query_start;
```

```shell
# 断开数据库连接
select pg_terminate_backend(pid) from (select pid from pg_stat_activity where datname = 'db_name' ) as a;

# 创建一个数据库归属其他用户
create database 'db_name' OWNER db_user;

# 查看数据库连接数
select count(*) from pg_stat_activity;

select * from pg_stat_activity;

# 查询数据库最大连接数，默认是100
postgres=> show max_connections;

# docker 部署到数据库修改最大连接数
# 进入容器，修改最大连接数
root@ip142:~# docker exec -ti postgres10-5433 bash
root@568a83e098a5:/# sed  -ri   '/max_connections/c max_connections = 2000' /var/lib/postgresql/data/postgresql.conf
```

用户只读权限设置：https://blog.csdn.net/qq_41018743/article/details/105492884

```sql
-- 以super user创建只读用户
CREATE USER readonly WITH PASSWORD '*****';

-- 以super user设置用户默认事务只读
alter user readonly set default_transaction_read_only=on;


-- 使用数据库的创建所有者去执行以下操作
-- 增加连接数据库权限
GRANT CONNECT ON DATABASE testDB to readonly;

-- 切换到 testDB
\c testDB;

-- 赋予用户权限访问public模式
GRANT USAGE ON SCHEMA public to readonly;

-- 赋予表序列查看权限
GRANT SELECT ON ALL SEQUENCES IN SCHEMA public TO readonly;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO readonly;


-- 新增加的表都默认增加权限
alter default privileges in schema public grant select on tables to readonly; 

-- 新增加的序列都默认增加权限
alter default privileges in schema public grant select on SEQUENCES to readonly;


```



## 三、从库配置

参考链接：https://help.aliyun.com/document_detail/53096.html

### 1、配置PostgreSQL主节点

1）输入以下SQL语句创建数据库账号replica，并设置密码及登录权限和备份权限。

本示例中将密码设置为`replica`。

```sql
CREATE ROLE replica login replication encrypted password 'replica';
CREATE ROLE replica login replication encrypted password 'rRweb9iojLxhVeWNddddddddddd';
```

2）修改配置文件

`data/pg_hba.conf`

在`IPv4 local connections`段添加下面两行内容。

```python
host    all             all             <从节点的VPC IPv4网段>          md5     #允许VPC网段中md5密码认证连接
host    replication     replica         <从节点的VPC IPv4网段>          md5     #允许用户从replication数据库进行数据同步
```

`postgresql.conf`

```shell
listen_addresses = '*'   #监听的IP地址
wal_level = hot_standby  #启用热备模式
synchronous_commit = on  #开启同步复制
max_wal_senders = 32     #同步最大的进程数量
wal_sender_timeout = 60s #流复制主机发送数据的超时时间
max_connections = 100    #最大连接数，从库的max_connections必须要大于主库的
```

修改完后重启服务



### 2、从节点上操作

参考链接：https://blog.51cto.com/u_13482808/6875114

```bash
pg_basebackup --help 
用法：
  pg_basebackup [选项] ...

控制输出的选项：
  -D, --pgdata=DIRECTORY 接收基本备份到目录，如果不存在会自动创建
  -F, --format=p|t       输出格式（plain,直接拷贝数据文件，tar 配合 -z -Z 进行打包压缩）
  -r, --max-rate=RATE         传输数据目录的最大传输速率（以 kB/s 为单位，或使用后缀“k”或“M”）
  -R,--write-recovery-conf 是否输出recovery-conf文件，方便后续使用备份快速搭建出从节点
  -T, --tablespace-mapping=OLDDIR=NEWDIR 将 OLDDIR 中的表空间重定位到 NEWDIR
      --waldir=WALDIR             预写日志目录的位置
  -X, --wal-method=none|fetch|stream 包含指定方法所需的 WAL 文件
  -z, --gzip                   压缩 tar 输出
  -Z, --compress=0-9    使用给定的压缩级别压缩 tar 输出

常规选项：
  -c, --checkpoint=fast|spread 设置快速或扩展检查点
  -C, --create-slot   创建复制槽
  -l, --label=LABEL  设置备份标签
  -n, --no-clean     出错后不清理
  -N, --no-sync     不等待更改安全写入磁盘
  -P, --progress    显示进度信息
  -S, --slot=SLOTNAME 要使用的复制槽
  -v, --verbose 输出详细信息
  -V, --version 输出版本信息，然后退出
      --no-slot 防止创建临时复制槽
      --no-verify-checksums 不验证校验和
  -?, --help 显示此帮助，然后退出

连接选项：
  -d, --dbname  数据库名称
  -h, --host      数据库服务器主机ip或套接字目录
  -p, --port      数据库口号
  -s, --status-interval=状态包发送到服务器的间隔时间（以秒为单位）
  -U, --username 连接用户，要有super权限
  -w, --no-password 从不提示输入密码
```



1）备份数据

使用pg_basebackup基础备份工具指定备份目录。

```shell
#保持pg data目录格式
pg_basebackup -h 192.168.1.1 -p 5432 -U replica -D /data/test -cfast -Xs -Pv

# 自动创建recovery.conf 
pg_basebackup -h 192.168.1.1 -p 5432 -U replica -D /data/test -cfast -Xs -Pv -R

# 打包成tar文件
pg_basebackup -h 192.168.1.1 -p 5432 -U replica -D /data/test -cfast -Xs -Pv -Ft

# 打包成tar.gz文件
pg_basebackup -h 192.168.1.1 -p 5432 -U replica -D /data/test -cfast -Xs -Pv -Ft -z 


```

新建并修改recovery.conf配置文件。

```shell
vim /var/lib/pgsql/11/data/recovery.conf

####分别找到以下参数，并将参数修改为以下内容：
standby_mode = on     #声明此节点为从库
primary_conninfo = 'host=<主节点IP> port=5432 user=replica password=replica' #对应主库的连接信息
recovery_target_timeline = 'latest' #流复制同步到最新的数据
```

修改postgresql.conf文件

```shell
max_connections = 1000             # 最大连接数，从节点需设置比主节点大
hot_standby = on                   # 开启热备
max_standby_streaming_delay = 30s  # 数据流备份的最大延迟时间
wal_receiver_status_interval = 5s  # 从节点向主节点报告自身状态的最长间隔时间
hot_standby_feedback = on          # 如果有错误的数据复制向主进行反馈
```

修改数据目录的权限

```
chown -R postgres.postgres /var/lib/pgsql/11/data
```

启动服务



### 3、验证

1）在主节点中运行以下命令，查看sender进程。

```undefined
ps aux |grep sender
```

返回结果如下，表示可成功查看到sender进程。

```yaml
postgres  2916  0.0  0.3 340388  3220 ?        Ss   15:38   0:00 postgres: wal sender     process replica 192.168.**.**(49640) streaming 0/F01C1A8
```

2）在从节点中运行以下命令，查看receiver进程。

```undefined
ps aux |grep receiver
```

返回结果如下，表示可成功查看到receiver进程。

```yaml
postgres 23284  0.0  0.3 387100  3444 ?        Ss   16:04   0:00 postgres: wal receiver process   streaming 0/F01C1A8
```

3）在主节点中进入PostgreSQL交互终端，输入以下SQL语句，在主库中查看从库状态。

```sql
select * from pg_stat_replication;
```

返回结果如下，表示可成功查看到从库状态。

```sql
pid | usesysid | usename | application_name | client_addr | client_hostname | client_port | backend_start | backend_xmin | state | sent_location | write_locati
on | flush_location | replay_location | sync_priority | sync_state 
------+----------+---------+------------------+---------------+-----------------+------------- +-------------------------------+--------------+-----------+---------------+-------------
---+----------------+-----------------+---------------+------------
2916 | 16393 | replica | walreceiver | 192.168.**.** | | 49640 | 2017-05-02 15:38:06.188988+08 | 1836 | streaming | 0/F01C0C8 | 0/F01C0C8 
| 0/F01C0C8 | 0/F01C0C8 | 0 | async
(1 rows)
```



### 4、查看主从延迟

如果主库没有插入或者修改的数据的sql执行，主从同步的延时会逐渐增加

```sql
select now() - pg_last_xact_replay_timestamp() AS replication_delay;
```



