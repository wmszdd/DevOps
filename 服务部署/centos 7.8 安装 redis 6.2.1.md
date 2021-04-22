# 介绍

Redis 是一个开源（BSD许可）的，内存中的数据结构存储系统，它可以用作数据库、缓存和消息中间件。 它支持多种类型的数据结构，如 字符串（strings）， 散列（hashes）， 列表（lists）， 集合（sets）， 有序集合（sorted sets） 与范围查询， bitmaps， hyperloglogs 和 地理空间（geospatial） 索引半径查询。 Redis 内置了 复制（replication），LUA脚本（Lua scripting）， LRU驱动事件（LRU eviction），事务（transactions） 和不同级别的 磁盘持久化（persistence）， 并通过 Redis哨兵（Sentinel）和自动 分区（Cluster）提供高可用性（high availability）。

# 安装

## 1.下载Redis安装包

```bash
[root@local ~]# wget https://download.redis.io/releases/redis-6.2.1.tar.gz
```

## 2.解压tar包

```bash
[root@local ~]# tar xzf redis-6.2.1.tar.gz
```
## 3.`cd`到解压目录

```bash
[root@local ~]# cd redis-6.2.1
```

## 4.安装gcc编译工具（如安装请忽略）

```bash
[root@local redis-6.2.1]# yum install -y gcc
```

## 5.编译

```bash
[root@local redis-6.2.1]# make && make install
```

## 6.启动Redis（先切换到src目录）

```bash
[root@local redis-6.2.1]# cd src/
```

### 直接启动

```bash
[root@local src]# ./redis-server
[root@localhost src]# ./redis-server
6347:C 08 Apr 2021 04:52:51.226 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
6347:C 08 Apr 2021 04:52:51.226 # Redis version=6.2.1, bits=64, commit=00000000, modified=0, pid=6347, just started
6347:C 08 Apr 2021 04:52:51.226 # Warning: no config file specified, using the default config. In order to specify a config file use ./redis-server /path/to/redis.conf
6347:M 08 Apr 2021 04:52:51.226 * Increased maximum number of open files to 10032 (it was originally set to 1024).
6347:M 08 Apr 2021 04:52:51.226 * monotonic clock: POSIX clock_gettime
                _._
           _.-``__ ''-._
      _.-``    `.  `_.  ''-._           Redis 6.2.1 (00000000/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._
 (    '      ,       .-`  | `,    )     Running in standalone mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
 |    `-._   `._    /     _.-'    |     PID: 6347
  `-._    `-._  `-./  _.-'    _.-'
 |`-._`-._    `-.__.-'    _.-'_.-'|
 |    `-._`-._        _.-'_.-'    |           http://redis.io
  `-._    `-._`-.__.-'_.-'    _.-'
 |`-._`-._    `-.__.-'    _.-'_.-'|
 |    `-._`-._        _.-'_.-'    |
  `-._    `-._`-.__.-'_.-'    _.-'
      `-._    `-.__.-'    _.-'
          `-._        _.-'
              `-.__.-'

6347:M 08 Apr 2021 04:52:51.227 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
6347:M 08 Apr 2021 04:52:51.227 # Server initialized
6347:M 08 Apr 2021 04:52:51.227 # WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
6347:M 08 Apr 2021 04:52:51.227 * Ready to accept connections
```

如上图：Redis启动成功，但是这种启动方式需要一直打开窗口，不能进行其他操作，我们可以一后台进程的方式启动

### 以后台进程启动Redis

- 修改redis.conf文件

```bash
[root@local redis-6.2.1]# vim redis.conf

# By default Redis does not run as a daemon. Use 'yes' if you need it.
# Note that Redis will write a pid file in /var/run/redis.pid when daemonized.
# When Redis is supervised by upstart or systemd, this parameter has no impact.
daemonize yes   #将no修改为yes
```

- 指定redis.conf文件启动

```bash
[root@local src]# ./redis-server /root/redis-6.2.1/redis.conf
```

- 使用`ps -aux`查看redis进程

```bash
[root@local src]# ps -aux | grep redis
root       6591  0.3  0.2 162500  9888 ?        Ssl  05:21   0:00 ./redis-server 127.0.0.1:6379
```

### 设置redis开机自启

在`/etc`目录下新建redis目录

```bash
[root@local etc]# mkdir redis
```

将`redis.conf`拷贝一份到`/etc/redis`目录下，并命名为`6379.conf`

```bash
[root@local redis]# cp /root/redis-6.2.1/redis.conf /etc/redis/6379.conf
```

将redis的启动脚本拷贝一份到`/etc/init.d`目录下

```bash
[root@local ~]# cp /root/redis-6.2.1/utils/redis_init_script /etc/init.d/redisd
```

设置redis开机自启动（先切换到`/etc/init.d`目录）

```bash
[root@local init.d]# chkconfig redisd on
```

现在可以直接已服务的形式启动和关闭redis了

- 启动：

```bash
[root@local init.d]# service redisd start
Starting Redis server...
```

- 关闭：

```bash
[root@local ~]# service redisd stop
Stopping ...
Waiting for Redis to shutdown ...
Redis stopped
```

**如果出现如下问题：**

```bash
[root@local redis-6.2.1]# service redisd start
/var/run/redis_6379.pid exists, process is already running or crashed
```
引起这类问题一般都是强制关掉电源或断电造成的，也是没等linux正常关机

科学处理的方法有两种：

1. 可用安装文件启动 `redis-server /etc/redis/6379.conf`
2. 软重启让系统自动恢复下就行了 `shutdown -r now`