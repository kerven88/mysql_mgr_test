# mysql_mgr_test
使用该脚本可以在同一个机器上快速搭建MYSQL MGR组复制。


1. 首先需要下载MySQL软件，配置/etc/hosts文件，下载二进制包都不需要什么安装了，直接解压放入指定的目录即可，比如/usr/local/mysql。
```
wget http://ftp.ntu.edu.tw/MySQL/Downloads/MySQL-5.7/mysql-5.7.32-el7-x86_64.tar.gz
tar zxf mysql-5.7.32-el7-x86_64.tar.gz -C /opt
mv /opt/mysql-5.7.32-el7-x86_64 /opt/mysql_5.7.32
```

假设 10.57.4.10 是服务器的IP，那么在/etc/hosts里面就尤其需要注意，把它务必配置好。  
比如下面的/etc/hosts的文件内容：
```
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

10.57.4.10  mysqltestdb
```

2. 有一个统一的配置文件 auto.cnf ,在这个配置文件里配置MySQL软件的路径，数据文件的路径即可。  
这些没有固定的内容，都是根据你的需求和具体的配置来定。比如auto.cnf的内容如下：
```
export base_dir=/opt/mysql_5.7.32   #配置为上面.tar.gz包解压出来的目录
export base_data_dir=/data/mysql    #数据目录
```

```
mkdir -p /data/mysql/{s1,s2,s3,s4,s5}
useradd mysql
chown -R mysql.mysql /opt/mysql_5.7.32
chown -R mysql.mysql /data/mysql
```

3. 配置节点列表，这是MGR部署关键的一个配置文件了。  
每个节点的配置分为4部分：节点的端口，节点的别名，节点的内部端口，节点的角色。  
节点的端口是数据库提供数据访问的端口，节点的别名，因为是在同一台服务器上模拟测试，所以需要标识不同节点的名字。  
节点的内部端口，这是MGR在各个节点之间的通信端口，最后是节点的角色，如果为Y就是提供读写权限，负责，只有读权限。  
如果是单主模式，最后的标识位第一个是Y,其他都为N  
```
24801 s1  24901 Y
24802 s2  24902 N 
24803 s3  24903 N 
24804 s4  24904 N
24805 s5  24905 N
```

如果是多主模式，则节点的角色都要标记为Y
```
24801 s1  24901 Y
24802 s2  24902 Y 
24803 s3  24903 Y 
24804 s4  24904 Y
24805 s5  24905 Y 
```

4. 运行脚本init.sh 不需要输入任何的参数。
这是最耗时的步骤，也是最核心的脚本。
```
yum install -y libaio
sh init.sh
```


5. 使用check_node.sh 脚本可以检查各个节点的状态，，输入参数为节点别名，比如s1  
   使用start_node.sh 脚本可以启动指定的节点,，输入参数为节点别名，比如s1  
   使用reset_node.sh 脚本可以在节点需要重新加入集群的时候使用，输入参数为节点别名，比如s1  
   使用stop_node.sh  脚本可以停止指定的节点，输入参数为节点别名，比如s1  
   使用conn_node.sh  脚本可以连接到指定的节点，输入参数为节点别名，比如s1     

比如我要检查节点s2的状态，是否为oline,是否应用数据正常，可以使用check_node.sh来查看。
```
sh check_node.sh s2
```

连接某个mysql实例
```
sh conn_node.sh s3
```


6. 感谢使用，有问题反馈，可以提交issue或者邮件给我jeanrock@126.com
