# 安装流程

下载RPM包

```bash
wget https://dev.mysql.com/get/mysql80-community-release-el7-1.noarch.rpm
```

本地安装RPM

```bash
yum localinstall mysql80-community-release-el7-1.noarch.rpm
```

更新YUM源

```bash
yum clean all
```

进行编译

```bash
makecache
```

# 创建mysql用户和组

创建mysql用户组

```bash
groupadd mysql
```

新建mysql用户并加入到mysql用户组

```bash
useradd -g mysql mysql
```

安装mysql

```bash
yum install mysql-community-server
```

# 启动mysql

```bash
systemctl start mysqld
```

查看初始密码

```bash
cat /var/log/mysqld.log | grep password
```

登录mysql

```bash
mysql -u root -p
```

修改初始化密码(密码复杂度有要求)

```sql
ALTER USER 'root'@'localhost' IDENTIFIED BY 'Sn888.com';
```

远程登陆设置

```mysql
use mysql;
```

设置所有用户可以连接

```sql
update user set host='%' where user='root';
```

刷新权限

```sql
FLUSH PRIVILEGES;
```
用工具测试发现已经可以正常连接

允许任何主机访问数据库

```sql
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%'WITH GRANT OPTION;
```

报错:`You are not allowed to create a user with GRANT`
原因:mysql8.0以前的版本可以使用grant在授权的时候隐式的创建用户，8.0以后已经不支持，所以必须先创建用户，然后再授权，解决办法如下：

使用mysql 数据库

```sql
use mysql;
```

特定用户的host 修改

```sql
update user set host='%' where user='root';
```

指定用户的授权

```sql
grant all privileges on test.* to root@'%';
```

刷新权限

```sql
FLUSH PRIVILEGES;
```

切换

```sql
use mysql;
```

查看

```sql
select user,host from user;
```

# 案例展示

允许szx用户使用oH!DytmzSa!^7dgf密码从任何主机连接到mysql服务器

```sql
GRANT ALL PRIVILEGES ON *.* TO 'szx'@'%'IDENTIFIED BY 'oH!DytmzSa!^7dgf' WITH GRANT OPTION;
```

允许用户szx从ip为192.168.1.1的主机连接到mysql服务器，并使用oH!DytmzSa!^7dgf作为密码

```sql
GRANT ALL PRIVILEGES ON *.* TO 'szx'@'192.168.1.1'IDENTIFIED BY 'oH!DytmzSa!^7dgf' WITH GRANT OPTION;
```