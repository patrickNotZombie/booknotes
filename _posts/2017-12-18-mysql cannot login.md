---
layout:     post
title:      "MySQL登陆不了的问题"
subtitle:   "root账号和新建用户的解决"
date:       2017-12-18 12:00:00
author:     "Patrick"
header-img: "img/mysql1025.jpg"
catalog:    true
tags:
    - MySQL
    - Linux
---



#MySQL登陆不了的问题解决


###1.root账号忘记密码

	解决办法就是先允许匿名登陆，然后进去数据库设置密码，然后删除匿名用户

####1.1更改my.cnf配置文件

```
vim /etc/my.cnf 或者 vi /etc/my.cnf
```

	在[mysqld]下添加skip-grant-tables
	然后重启mysql服务即可匿名登陆

####1.2更改root用户密码

```
MySQL> UPDATE mysql.user SET Password=PASSWORD('新密码') where USER='root';
MySQL> flush privileges; 
```

####1.3恢复my.cnf配置文件

	把/etc/my.cnf中的skip-grant-tables注释掉，然后重启mysq
**记得重新登录后删除匿名用户，不然会遇到下面的问题2，同时不删除匿名用户极不安全**

###2.新建用户无法登陆

增加普通用户后无法登陆可尝试以下方法，执行：
 
```
mysql> use mysql 
mysql> delete from user where user=''; 
mysql> flush privileges; 
```

删除匿名用户就可以正常登陆。