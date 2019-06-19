---
title: 在Ubuntu 18.04系统上安装MariaDB 10.4的步骤
categories:
  - Ubuntu
tags:
  - Ubuntu
  - MariaDB
date: 2019-06-19 21:10:44
---

## 在 Ubuntu 18.04 上安装MariaDB 10.4 的具体步骤

要在 `Ubuntu 18.04` 上安装最新版的 `MariaDB`，需要将 `MariaDB` 存储库添加到系统中。

### 第一步、安装 `software-properties-common`，如果缺少则进行安装：
```bash
sudo apt-get install software-properties-common
```

### 第二步、导入 MariaDB gpg 密钥：

运行以下命令将 `Repository Key` 添加到系统：
```bash
sudo apt-key adv --recv-keys --keyserver hkp://keyserver.ubuntu.com:80 0xF1656F24C74CD1D8
```

### 第三步、添加apt存储库

导入PGP密钥后，继续添加存储库URL：
```bash
sudo add-apt-repository "deb [arch=amd64,arm64,ppc64el] http://mariadb.mirror.liquidtelecom.com/repo/10.4/ubuntu $(lsb_release -cs) main"
```

如果系统中没有 add-apt-repository ，请安装，参考Ubuntu 18.04/16.04/Debian 9上安装 add-apt-repository 的方法。

### 第四步、在Ubuntu 18.04上安装MariaDB Server

最后一步是安装MariaDB Server：
```bash
sudo apt-get update
sudo apt-get install mariadb-server mariadb-client
```

## 初始化 MariaDB 10.4.6
默认情况下，新安装的 `Mariadb` 的密码为空，执行非管理没权限的 mysql 命令可以出现异常：
```bash
mysql -u root -p

Enter password:
ERROR 1698 (28000): Access denied for user 'root'@'localhost'
```

这个时候改成使用 sudo 的 mysql 虽然也能登陆数据库，但是从其他客户端使用普通账号连接还是有问题。
所以，如果是刚安装第一次使用，请使用 `mysql_secure_installation` 命令初始化。
```bash
# mysql_secure_installation

NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MariaDB to secure it, we'll need the current
password for the root user.  If you've just installed MariaDB, and
you haven't set the root password yet, the password will be blank,
so you should just press enter here.

Enter current password for root (enter for none):
OK, successfully used password, moving on...

Setting the root password ensures that nobody can log into the MariaDB
root user without the proper authorisation.

Set root password? [Y/n] y
New password:
Re-enter new password:
Password updated successfully!
Reloading privilege tables..
 ... Success!


By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n] y
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] y
 ... Success!

By default, MariaDB comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n] y
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n] y
 ... Success!

Cleaning up...

All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!
```

## 设置 MariaDB 的远程连接

MariaDB 初始化成功之后，登录到MariaDB数据库：

```bash
mysql -u root -p
```

设置当前数据库：

```bash
MariaDB [(none)]> use mysql;

```

配置所有电脑可以通过 root:123456 访问数据库：

```bash
MariaDB [(mysql)]> GRANT ALL PRIVILEGES ON *.* to 'root'@'%' identified by '123456';
```

从mysql数据库中的授权表重新载入权限

```bash
MariaDB [(mysql)]> flush privileges;
```

编辑 MariaDB 10.4 的配置文件：/etc/mysql/my.cnf， MySQL的为 /etc/mysql/mysql.conf.d/mysqld.cnf，
找到 "bind-address = 127.0.0.1" , 这一行要注释掉。

重启mysql服务 命令：

```bash
sudo service mysql restart
```

## 设置默认字符集
 
此时使用 insert into 语句插入包含中文内容的字段时，会出现 `Error 1366 - Incorrect integer value` 错误，在 mysql 中查看字符集设置：
```bash
MariaDB [(none)]>  SHOW VARIABLES LIKE 'character%';
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | utf8                       |
| character_set_connection | utf8                       |
| character_set_database   | latin1                     |
| character_set_filesystem | binary                     |
| character_set_results    | utf8                       |
| character_set_server     | latin1                     |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
```

结果显示 `character_set_database` 和 `character_set_server` 的值为 `latin1`，不支持中文（包含其他多字节文字）。
解决办法：修改 /etc/mysql/my.cnf 

打开 my.cnf 后，在文件内的相应的位置增加如下几行代码:
```editorconfig	
[client]
default-character-set=utf8mb4
[mysqld]
character-set-server=utf8mb4
[mysql]
default-character-set=utf8mb4
```
保存并退出。

重启mysql服务

```bash
service mysql restart
```

重新进入 mysql，查看数据库服务器字符集信息：
```bash
MariaDB [(none)]> SHOW VARIABLES LIKE 'character%';
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | utf8mb4                    |
| character_set_connection | utf8mb4                    |
| character_set_database   | utf8mb4                    |
| character_set_filesystem | binary                     |
| character_set_results    | utf8mb4                    |
| character_set_server     | utf8mb4                    |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
8 rows in set (0.002 sec)
```

新建数据库，执行 insert into 语句测试添加记录。