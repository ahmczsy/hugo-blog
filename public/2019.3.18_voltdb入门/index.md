# VoltDB入门


# 安装运行

### 环境要求

* python2.7以上，不支持python3

* java8

### 安装    

* 到官网注册个账号才能把下载链接发到邮箱，很蛋疼，不能直接下载 

* 解压

* 把voltdb/bin/加入环境变量
<!--more-->
### 运行
* 初始化工作空间

  ```shell
  voltdb inti  
  ```
  在当前目录生成voltdbroot文件夹，里面存放着数据库数据之类的东西，还有命令行快照，用来恢复数据

* 运行voltdb
  ```shell
  voltdb start
  ```

* 通过cmd操作数据库
  ```shell
  sqlcmd
  ```
  用过sqlcmd进入命令行,和MySQL的命令行差不多

* 关闭
  ```shell
  voltadmin shutdown
  ```
# Java连接VoltDB

* 驱动jar
    ```xml
    <!-- https://mvnrepository.com/artifact/org.voltdb/voltdbclient -->
    <dependency>
        <groupId>org.voltdb</groupId>
        <artifactId>voltdbclient</artifactId>
        <version>8.4.1</version>
    </dependency>
    ```

* 传统的jdbc方式连接VoltDB
  ```java
    String driver = "org.voltdb.jdbc.Driver";
    String url = "jdbc:voltdb://host:port?jdbc.committhrowexception=false&jdbc.rollbackthrowexception=false&autoreconnect=true";
    Class.forName(driver);
    this.connection = DriverManager.getConnection(url);
  ```
  因为voltdb不支持传统的事务方式，所以在jdbc添加两个参数使在使用事务时不报异常

* 使用client来连接VoltDB
  ```java
  ClientConfig config = new ClientConfig();
        this.client = ClientFactory.createClient(config);
        client.createConnection("192.168.1.241");
  ```
  如果使用这种方式来操作数据库，所有的操作只能通过存储过程来实现

# 简单性能测试

* 启动时占用550M左右的内存
* 每秒45次插入时,CPU占用大概20%
* 每秒3900次插入时,CPU满载,没有数据丢失
* 对100W单表的一个字段进行like查询,耗时0.2S
* 三百万条下图类似的数据，加上voltdb本身的进程大概占用1.22G内存
 ![Am5FAS.png](https://s2.ax1x.com/2019/03/18/Am5FAS.png)


# 其他

* voltdb是内存关系数据库,所以查询比redis方便

* 不支持beganTransaction autoCommit之类的操作,要想进行事务相关的操作,只能使用存储过程

* 不支持外键,自增字段之类的

* sqlcmd可以执行sql脚本

  ```shell
  sqlcmd < test.sql
  ```
# 参考

* https://docs.voltdb.com/UsingVoltDB/ddlref_createtable.php
* https://docs.voltdb.com/
