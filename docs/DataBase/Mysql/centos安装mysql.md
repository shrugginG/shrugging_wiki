## 利用RPM包安装

### 检测是否存在历史或者自带mysql

1. 查看是否存在历史mysql

   ```shell
   rpm -qa|grep mariadb
   ```

2. 如果存在使用rpm命令卸载

   ```shell
   sudo rpm -e --nodeps mariadb-libs
   ```

   

### 安装mysql

1. import `mysql-5.7.28-1.el7.x86_64.rpm-bundle.tar`to `/opt/software`

2. tar -xf mysql-5.7.28-1.el7.x86_64.rpmbundle.tar

3. 执行rpm安装

   ```shell
   sudo rpm -ivh mysql-community-common-5.7.28-1.el7.x86_64.rpm
   sudo rpm -ivh mysql-community-libs-5.7.28-1.el7.x86_64.rpm
   sudo rpm -ivh mysql-community-libs-compat-5.7.28-1.el7.x86_64.rpm
   sudo rpm -ivh mysql-community-client-5.7.28-1.el7.x86_64.rpm
   sudo rpm -ivh mysql-community-server-5.7.28-1.el7.x86_64.rpm
   ```

   注意:按照顺序依次执行 如果 Linux 是最小化安装的，在安装 mysql-community-server-5.7.28-1.el7.x86_64.rpm 时 可能会出现如下错误: 

   ```shell
   [shrugging@VM-12-3-centos mysql]$ sudo rpm -ivh mysql-community-server5.7.28-1.el7.x86_64.rpm 
   警告：mysql-community-server-5.7.28-1.el7.x86_64.rpm: 头 V3 DSA/SHA1  Signature, 密钥 ID 5072e1f5: NOKEY错误：依赖检测失败： libaio.so.1()(64bit) 被 mysql-community-server-5.7.28-1.el7.x86_64  需要 libaio.so.1(LIBAIO_0.1)(64bit) 被 mysql-community-server-5.7.28- 1.el7.x86_64 需要 libaio.so.1(LIBAIO_0.4)(64bit) 被 mysql-community-server-5.7.28- 1.el7.x86_64 需要
   ```

   通过 yum 安装缺少的依赖,然后重新安装 mysql-community-server-5.7.28-1.el7.x86_64 即可 

   ```shell
   [shrugging@VM-12-3-centos mysql]$ yum install -y libaio
   ```

4. 删除/etc/my.cnf 文件中 datadir 指向的目录下的所有内容,如果有内容的情况下: 查看 datadir 的值： 

   ```shell
   [mysqld] 
   datadir=/var/lib/mysql
   ```

    删除/var/lib/mysql 目录下的所有内容: 

   ```shell
   [shrugging@VM-12-3-centos mysql]$ cd /var/lib/mysql 
   [shrugging@VM-12-3-centos mysql]$ sudo rm -rf ./* //注意执行命令的位置
   ```

5. 初始化数据库 

   ```shell
   [shrugging@VM-12-3-centos mysql]$ sudo mysqld --initialize --user=mysql
   ```

6. 查看临时生成的 root 用户的密码 

   ```shell
   [shrugging@VM-12-3-centos mysql]$ sudo cat /var/log/mysqld.log
   ```
   
   ![image-20220421183657812](https://images.shrugging.cn/image-20220421183657812.png)

7. 启动 MySQL 服务 

   ```shell
   [atguigu @hadoop102 opt]$ sudo systemctl start mysql
   ```

8. 登录 MySQL 数据库 

   ```shell
   [atguigu @hadoop102 opt]$ mysql -uroot -p Enter password: 输入临时生成的密码
   ```

9. 必须先修改 root 用户的密码,否则执行其他的操作会报错 

   ```shell
   mysql> set password = password("新密码");
   ```

10. 修改 mysql 库下的 user 表中的 root 用户允许任意 ip 连接

    ```shell
    mysql> update mysql.user set host='%' where user='root'; mysql> flush privileges;
    ```

    
