title: 修改最新版本Mysql默认密码(v5.7.10)
date: 2016-02-02 13:18:17
tags:
- 其它
- Mysql
---

MySql版本: v5.7.10

## Step1
安装mysql，加入PATH。

```bash
> vim ~/.zshrc
```
修改如下内容：注意路径中间用`:`隔开
export PATH="/usr/local/other_path:/usr/local/mysql/bin/:$PATH"

## Step2
在系统偏好，最下面的mysql中点击`Stop MySql Server`按钮，关闭mysql后台进程

## Step3
跳过mysql权限认证机制

```bash
> sudo mysqld_safe --user=mysql --skip-grant-tables --skip-networking
```
启动了mysql的安全模式，可以打开新的命令行窗口进入mysql

## Step4
登录mysql，修改密码

```bash
> mysql
> use user;
> update user set authentication_string=password('123456') where user='root';
> flush privileges;
> exit
```
注：mysql5.7以后用户的password字段名变成了authentication_string，这点很坑。

## Step5
关闭相关进程，重新登陆mysql

```bash
> ps -A | grep mysql
2004 ttys000    0:00.00 grep --color=auto --exclude-dir=.bzr --exclude-dir=CVS --exclude-dir=.git --exclude-dir=.hg --exclude-dir=.svn mysql
1733 ttys001    0:00.02 sudo mysqld_safe --user=mysql --skip-grant-tables --skip-networking
1735 ttys001    0:00.02 /bin/sh /usr/local/mysql/bin//mysqld_safe --user=mysql --skip-grant-tables --skip-networking
1846 ttys001    0:00.37 /usr/local/mysql/bin/mysqld --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data --plugin-dir=/usr/local/mysql/lib/plugin --user=mysql --skip-grant-tables --skip-networking --log-error=/usr/local/mysql/data/wangchaoyouzancomdeMacBook-Pro.local.err --pid-file=/usr/local/mysql/data/wangchaoyouzancomdeMacBook-Pro.local.pid
> sudo kill -9 2004 1733 1735 1846
```

重新在系统偏好的Mysql中启动mysql

```bash
> mysql -u root -p
Enter password:
123456

Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 39
Server version: 5.7.10

Copyright (c) 2000, 2015, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```

## Step6
客户端用root用户登录失败提示如下内容：
> Your password has expired. To log in you must change it using a client that supports expired passwords. 
> -- Navicat Premium for MySql

> You must reset your password using ALTER USER statement before executing this statement.
> -- MySql Workbentch

其中workbentch会自动跳出新窗口让你重置密码。
Navicat不会弹出窗口就用命令行的方式重置：

```bash
> mysql -u root -p
Enter password:


mysql> alter user 'root'@'localhost' identified by 'admin' password expire never;
mysql> exit
```
[官方Alter User文档](http://dev.mysql.com/doc/refman/5.7/en/alter-user.html)

密码的过期策略有四种选项
1. EXPIRE 过期
2. EXPIRE DEFAULT 过期
3. EXPIRE NEVER 永不过期
4. EXPIRE INTERVAL N DAY n天后过期

```sql
password_option: {
	PASSWORD EXPIRE
	| PASSWORD EXPIRE DEFAULT
	| PASSWORD EXPIRE NEVER
	| PASSWORD EXPIRE INTERVAL N DAY
}
```

在navicat中修改连接密码，即可连接成功。

