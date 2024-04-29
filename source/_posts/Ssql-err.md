---
title: "mac系统中进入mysql遇到Access denied for user 'root'@'localhost"
date: '2019/03/20 14:59:21'
categories:
   - mysql
tags:
   - mysql
toc_level: 3
version: 3
---

#### [](about:blank#Access-denied-for-user-%E2%80%98root%E2%80%99-%E2%80%99localhost%E2%80%99-using-password-YES "Access denied for user ‘root’@’localhost’ (using password: YES)")Access denied for user ‘root’@’localhost’ (using password: YES)

1)Stop mysql (Kill mysql process or run following command) 停止mysql服务

```
sudo /usr/local/mysql/support-files/mysql.server stop
```

2.  Start it in safe mode 用安全模式进入mysql 这一步可以看成是不输入密码就进入mysql命令行

```
sudo mysqld_safe --skip-grant-tables
```

3.  Open another terminal and run the following command (Keep last terminal open) 在另一个窗口中进入mysql

```
mysql -u root
```

4.  Run the following command with suitable new password on the mysql console 修改密码

```
UPDATE mysql.user SET Password=PASSWORD('password') WHERE User='root';
```

5.  刷新一下缓存```
    FLUSH PRIVILEGES; 
    ```
    

  

