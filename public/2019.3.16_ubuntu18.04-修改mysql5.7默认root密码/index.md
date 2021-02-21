# Ubuntu18.04 修改Mysql5.7默认root密码


#### 步骤
* 设置mysql免密码登陆
  编辑/etc/mysql/my.cnf文件,在最后加入以下设置
```aidl
[mysqld]
skip-grant-tables=1
```
* 重启mysql
```aidl
$ sudo service mysql stop
$ sudo service mysql strat
```
<!--more-->

* 进入mysql，先修改验证方式，再改密码
```aidl
$ mysql
//修改验证方式
mysql> USE mysql;
mysql> UPDATE user SET plugin='mysql_native_password' WHERE User='root';
//修改密码
mysql> update mysql.user set authentication_string=password('123qwe') where user='root' and Host = 'localhost';
//刷新权限
mysql> flush privileges;
//推出
musql> exit;
```

* 重启mysql服务 登陆mysql
```aidl
$ mysql -uroot -p
```

#### 参考
https://stackoverflow.com/questions/39281594/error-1698-28000-access-denied-for-user-rootlocalhost#


