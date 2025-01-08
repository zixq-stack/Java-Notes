# 前言

> 可以采用组合式安装，如：https://oneinstack.com/ 选择好要安装的，然后复制安装命令就可以一键搞定很多东西了



# VMware安装Centos7

按照物理机CPU实际情况，选择处理器配置，处理器数量*每个处理器内存数量要小于等于物理机CPU的数量，否则报错

选择分配给虚拟机的内存，最少2G

磁盘类型选择SCSI

- IDE: 老的磁盘类型
- SCSI: 服务器上推荐使用的磁盘类型，串口。
- SATA: 也是串口，也是新的磁盘类型



启动虚拟机时，选择Install CentOS 7 ，然后回车即可，不要选择Test this media & install CentOS 7, 然后就没有然后了



分区划分：

- `/boot`：引导分区，建议给1G，设备类型为标准分区，文件系统为ext4
- `swap`：交换分区，建议设置与内存大小一致. 2G，设备类型为标准分区，文件系统为swap
- `/`：剩余的磁盘大小全部分配。 /为linux文件系统的根目录，设备类型为标准分区，文件系统为ext4





## 设置静态IP

> 使用 VirtualBox 安装的CentOS7，改静态IP [去这里](https://www.bilibili.com/video/BV1rT4y137vD/?spm_id_from=333.337.search-card.all.click&vd_source=e4988886491488544ea407864e95dfc1)
>
> PS：视频中改 `ifcfg-enp0s8` 文件的 IPADDR 时，该值需要参考VirtualBox开机页 管理->主机网络管理器->网卡（A）->IPv4的值。如该 IPv4 为 192.168.56.1，那么 `ifcfg-enp0s8` 中IPADDR的值 192.168.56.xxxx，该 xx 的值为 0 - 254 以内 即可

**查看自己虚拟机的NAT配置**

![image](https://img2023.cnblogs.com/blog/2421736/202405/2421736-20240504141832953-773870612.png)


**建议**：先把自己的NAT配置恢复默认一下，因为有时一开始给的ip、网关这两个有问题，特别是第一次进行配置时

![image](https://img2023.cnblogs.com/blog/2421736/202405/2421736-20240504141832931-524797062.png)


![image](https://img2023.cnblogs.com/blog/2421736/202405/2421736-20240504141833018-2105806070.png)



然后查看自己的NAT相关配置

![image](https://img2023.cnblogs.com/blog/2421736/202405/2421736-20240504141832998-309635047.png)


![image](https://img2023.cnblogs.com/blog/2421736/202405/2421736-20240504141832946-1885157463.png)




**进入到Linux中**，执行命令：`vim /etc/sysconfig/network-scripts/ifcfg-ens33`

> `ens33` 是你的linux的网卡名，使用 `ip addr` 可以先看一下

![](https://img2023.cnblogs.com/blog/2421736/202410/2421736-20241022011657900-1339903159.png)

文件内容编写如下：

```bash
TYPE="Ethernet"
PROXY_METHOD="none"
BROWSER_ONLY="no"
BOOTPROTO="static"    # 改为 static  即静态  dhcp是动态获取，每次会变
DEFROUTE="yes"
IPV4_FAILURE_FATAL="no"
IPV6INIT="yes"
IPV6_AUTOCONF="yes"
IPV6_DEFROUTE="yes"
IPV6_FAILURE_FATAL="no"
IPV6_ADDR_GEN_MODE="stable-privacy"
NAME="ens33"
UUID="07440d98-afc3-42d6-85e4-bd0603372106"
DEVICE="ens33"
ONBOOT="yes"    # 改为yes

# 下面内容就是刚刚让记得几个值
IPADDR="192.168.72.109"      # PADDR和刚刚记住的要在一个段中，如：100改成255以内即可，建议在100左右，这个值一般不会冲突
NETMASK="255.255.255.0"
GATEWAY="192.168.72.2"
DNS1="114.114.114.114"
```


对照图

![image](https://img2023.cnblogs.com/blog/2421736/202405/2421736-20240504141833295-1093228031.png)

然后保存




**重启网络**

```bash
systemctl restart network`
```

输入`ip addr`就会发现地址已经变成静态的了，也就可以使用SSH客户端工具输入ip、username、password就可以链接了（如：xshell、windTerm......）


![image](https://img2023.cnblogs.com/blog/2421736/202405/2421736-20240504141833203-1766431696.png)



# Linux中各目录解读

1. `/` (Root，根目录)：所有其他目录的根
2. `/bin` (Binaries，二进制文件)：包含基本命令行程序
3. `/boot` (Boot，启动)：包含启动系统所需的文件
4. `/dev` (Devices，设备)：包含设备文件
5. `/etc` (Etcetera，等等)：包含系统配置文件
6. `/home`：用户的家目录通常位于此，例如 `/home/username`
7. `/lib` (Library，库)：包含系统程序所需的库文件
8. `/media`：用于挂载媒体设备
9. `/mnt` (Mount，挂载)：用于临时挂载文件系统
10. `/opt` (Optional，可选)：包含额外的应用程序，即类似Windows中我们安装程序自定义目录所在地
11. `/proc` (Processes，进程)：包含系统进程的信息
12. `/root`：超级用户的家目录
13. `/sbin` (System Binaries，系统二进制文件)：包含系统级别的基本命令
14. `/srv` (Service，服务)：包含服务运行所需的数据
15. `/sys`：包含系统硬件信息和状态
16. `/tmp` (Temporary，临时)：包含临时文件
17. `/usr` (Unix Software Resource，Unix操作系统资源)：包含大量应用程序和文件，即类似Windows中的 C:\Program Files (x86)
18. `/var` (Variable，变量)：包含在运行时改变大小的文件，如日志文件



# CentOS 7和8 停止维护

yum命令已经不可以用了，重新拉取仓库信息即可:

1. https://developer.aliyun.com/mirror/centos?spm=a2c6h.13651102.0.0.3e221b11A9CAEo

2. [解决方式1](https://www.cnblogs.com/maowenqiang/articles/7728685.html)

3. [解决方式2](https://www.cnblogs.com/zengzuo613/p/18292809)

# Git工具安装

## 方法一：通过包管理器安装

对于CentOS 系统来讲，直接执⾏如下命令即可安装：

```bash
yum install git
```

> **提示**
>
> 通过这种⽅式安装的 Git 不是较新版的 Git





## 方法二：通过源码编译安装

**1、准备Git.xxxx.tar.gz安装包**：[Git安装包下载](https://github.com/git/git/tags)

**2、上传服务器，解压**：本人是放在`/root`下的

```bash
tar -zxvf v2.45.0.tar.gz
```

**3、安装需要的依赖**

```bash
yum install curl-devel expat-devel gettext-devel openssl-devel zlibdevel gcc-c++ perl-ExtUtils-MakeMaker
```

**4、编译安装Git**：进⼊到对应⽬录，执⾏配置、编译、安装命令

```bash
cd git-2.45.0/

# profix 改为自己要安装的路径
make profix=/usr/local/git

make install
```

**5、配置Git环境变量**

```bash
# 编写配置文件
vim /etc/profile


# 文件内容：尾部加⼊ Git 的 bin 路径配置即可
export GIT_HOME=/usr/local/git
export PATH=$PATH:$GIT_HOME/bin
```

最后执⾏ `source /etc/profile` 使环境变量⽣效即可

执⾏ `git --version` 可查看到安装后的版本即可







# JDK安装

> 这⾥安装的是Oracle JDK

**1、准备安装包**：可去官网下载，也可以去 [这里](http://www.codebaoku.com/jdk/jdk-oracle-jdk17.html)

**2、卸载系统已有的OPENJDK（如果有）**

```bash
# 查找已安装的OpenJDK包
rpm -qa | grep java


# 以将 java 开头的安装包均卸载
yum -y remove java-1.7.0-openjdk-1.7.0.141-2.6.10.5.el7.x86_64
yum -y remove java-1.8.0-openjdk-1.8.0.131-11.b12.el7.x86_64
...........省略........
```

**3、上传安装包到服务器**：本人路径为`/usr/local`

**4、解压、重命名**

```bash
# 解压
tar -zxvf jdk..........tar.gz


# 重命名
mv jdk........	jdk8
```

**5、配置JDK环境变量**

```bash
vim /etc/profile


# 在文件尾部加入如下内容	路径改为自己的
JAVA_HOME=/usr/local/jdk8
CLASSPATH=$JAVA_HOME/lib/
PATH=$PATH:$JAVA_HOME/bin
export PATH JAVA_HOME CLASSPATH
```

**6、让环境生效**

```bash
source /etc/profile
```

**7、验证**

```bash
java -version

javac
```









# Maven安装

**1、准备安装包**：[Maven安装包下载](https://maven.apache.org/download.cgi)

**2、上传服务器，解压**：本人上传路径为`/usr/local`

```bash
# 解压
tar -zxvf apache-maven.........tar.gz
```

**3、配置Maven加速镜像源**

这⾥配置的是阿⾥云的maven镜像源

编辑修改 `/usr/local/apache-maven-3.6.3/conf/settings.xml` 文件， 在`<mirrors></mirrors>` 标签对⾥添加如下内容即可：

```xml
<mirror>
    <id>alimaven</id>
    <name>aliyun maven</name>
    <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
    <mirrorOf>central</mirrorOf>
</mirror>
```

**4、配置环境变量**

```bash
# 编辑文件
vim /etc/profile

# 在文件尾部加入如下内容 	路径改为自己的
export MAVEN_HOME=/usr/local/apache-maven-3.6.3
export PATH=$MAVEN_HOME/bin:$PATH
```

**5、让环境生效**

```bash
source /etc/profile
```

**5、验证**

```bash
mvn -v
```





# MySQL安装

docker安装的方式：

```bash
docker run --name mysql80 -p 3306:3306 -v /usr/local/docker-mysql80/conf:/etc/mysql/conf.d -v /usr/local/docker-mysql80/datadir:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=072413mcs -d mysql:8.0.27 --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
```



**1、准备安装包**：[MySQL安装包官网下载](https://downloads.mysql.com/archives/community/)，一般选择MySQL57，新版尝鲜就可以了

**2、卸载系统自带的Mariadb（如果有）**

```bash
# 查看已安装的Mariadb
rpm -qa|grep mariadb


# 卸载
yum -y remove mariadb-server-5.5.56-2.el7.x86_64
yum -y remove mariadb-5.5.56-2.el7.x86_64
yum -y remove mariadb-devel-5.5.56-2.el7.x86_64
yum -y remove mariadb-libs-5.5.56-2.el7.x86_64
```

**3、上传安装包到服务器，解压，重命名**：本人路径为`/usr/local`

```bash
# 解压
tar -zxvf /mysql-5.7.30-linux-glibc2.12-x86_64.tar.gz

# 重命名
mv mysql-5.7.30-linux-glibc2.12-x86_64 mysql57
```

**4、创建MySQL用户与用户组**

```bash
groupadd mysql

useradd -g mysql mysql
```

同时新建 `/usr/local/mysql57/data` ⽬录，后续备⽤

```bash
mkdir -p /usr/local/mysql57/data
```

**5、修改MySQL目录的归属用户**

```bash
chown -R mysql:mysql /usr/local/mysql57
```

**6、编写配置文件**：在 `/etc` ⽬录下新建 `my.cnf` ⽂件，编写内容如下

```bash
[mysql]
# 设置mysql客户端默认字符集
default-character-set=utf8
socket=/var/lib/mysql57/mysql.sock
[mysqld]
skip-name-resolve
# 设置3306端⼝
port = 3306
socket=/var/lib/mysql57/mysql.sock
# 设置mysql的安装⽬录
basedir=/usr/local/mysql57
# 设置mysql数据库的数据的存放⽬录
datadir=/usr/local/mysql57/data
# 允许最⼤连接数
max_connections=200
# 服务端使⽤的字符集默认为8⽐特编码的latin1字符集
character-set-server=utf8
# 创建新表时将使⽤的默认存储引擎
default-storage-engine=INNODB
lower_case_table_names=1
max_allowed_packet=16M
```

同时创建上述配置中的 `/var/lib/mysql57` ⽬录，并修改权限

```bash
mkdir -p /var/lib/mysql57

chmod 777 /var/lib/mysql57
```

**7、安装MySQL**

```bash
# 进入MySQL解压目录
cd /usr/local/mysql57

# 安装	baseDir 和 datadir 改为自己的		下面这句指令执行完会出现 Root账号的密码
./bin/mysqld --initialize --user=mysql --basedir=/usr/local/mysql57 --datadir=/usr/local/mysql57/data
```

![image-20240504150354332](https://img2023.cnblogs.com/blog/2421736/202405/2421736-20240504150233170-1160112128.png)



> **重要提示**
>
> 记得把上述密码拷贝下来，后面改密码要用

**8、复制启动脚本到资源⽬录**

```bash
cp ./support-files/mysql.server /etc/init.d/mysqld
```

并修改 `/etc/init.d/mysqld `，修改其 `basedir `和 `datadir `为实际对应⽬录：

```bash
# 修改文件
vim /etc/init.d/mysqld

# 修改内容	这两个大概在文件底部，直接搜索即可
basedir=/usr/local/mysql
datadir=/usr/local/mysql/data
```

**9、设置MYSQL系统服务并开启自启**

- 增加 mysqld 服务控制脚本执行权限

```bash
chmod +x /etc/init.d/mysq
```

- 将 mysqld 服务加入到系统服务

```bash
chkconfig --add mysqld
```

- 检查 mysqld 服务是否已经生效

```bash
chkconfig --list mysqld
```

![image-20240504150903973](https://img2023.cnblogs.com/blog/2421736/202405/2421736-20240504150740575-815550226.png)



**10、启动mysqld**

```bash
service mysqld start
```

**11、配置环境变量**

```bash
# 编写文件
vim ~/.bash_profile


# 文件底部加入如下内容	路径改为自己的
export PATH=$PATH:/usr/local/mysql57/bin


# 让配置生效
source ~/.bash_profile
```

**12、修改Root账号密码**

```mysql
# 登录mysql	密码就是前面让拷贝的密码
mysql -u root -p


# 修改root账号的密码
alter user user() identified by "1234567";


# 刷新权限
flush privileges;
```

**13、设置支持远程连接**

```mysql
mysql> use mysql;
mysql> update user set user.Host='%' where user.User='root';
mysql> flush privileges;
```

最后利用连接工具测试一下就可以了





# Redis安装

## 单机安装

1. 安装需要的依赖

```shell
yum install -y gcc tcl
```

2. 上传压缩包并解压

```shell
tar -zxf redis-7.0.12.tar.gz
```

3. 进入解压的redis目录

```shell
cd redis-7.0.12
```

4. 编译并安装

```shell
make && make install
```

默认的安装路径是在 `/usr/local/bin`目录下

![image-20230723232738946](https://img2023.cnblogs.com/blog/2421736/202405/2421736-20240504151418714-1427743853.png)



该目录已经默认配置到环境变量，因此可以在任意目录下运行这些命令。其中：

- redis-cli：是redis提供的命令行客户端
- redis-server：是redis的服务端启动脚本
- redis-sentinel：是redis的哨兵启动脚本





## 启动Redis

redis的启动方式有很多种，例如：

- 默认启动
- 指定配置启动
- 开机自启





### 默认启动

安装完成后，在任意目录输入redis-server命令即可启动Redis：

```shell
redis-server
```

这种启动属于“前台启动”，会阻塞整个会话窗口，窗口关闭或者按下`CTRL + C`则Redis停止





### 指定配置启动

如果要让Redis以“后台”方式启动，则必须修改Redis配置文件，就在之前解压的redis安装包下（`/usr/local/src/redis-6.2.6`），名字叫redis.conf

修改redis.conf文件中的一些配置：可以先拷贝一份再修改

```properties
# 允许访问的地址，默认是127.0.0.1，会导致只能在本地访问。修改为0.0.0.0则可以在任意IP访问，生产环境不要设置为0.0.0.0
bind 0.0.0.0
# 守护进程，修改为yes后即可后台运行
daemonize yes
# 密码，设置后访问Redis必须输入密码
requirepass 072413
```



Redis的其它常见配置：

```properties
# 监听的端口
port 6379
# 工作目录，默认是当前目录，也就是运行redis-server时的命令，日志、持久化等文件会保存在这个目录
dir .
# 数据库数量，设置为1，代表只使用1个库，默认有16个库，编号0~15
databases 1
# 设置redis能够使用的最大内存
maxmemory 512mb
# 日志文件，默认为空，不记录日志，可以指定日志文件名
logfile "redis.log"
```



启动Redis：

```sh
# 进入redis安装目录 
cd /opt/redis-6.2.13
# 启动
redis-server redis.conf
```



停止服务：

```sh
# 利用redis-cli来执行 shutdown 命令，即可停止 Redis 服务，
# 因为之前配置了密码，因此需要通过 -u 来指定密码
redis-cli -u password shutdown
```





### 开机自启

可以通过配置来实现开机自启。

首先，新建一个系统服务文件：

```sh
vim /etc/systemd/system/redis.service
```

内容如下：

```shell
[Unit]
Description=redis-server
After=network.target

[Service]
Type=forking
ExecStart=/usr/local/bin/redis-server /opt/redis-7.0.12/redis.conf
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

然后重载系统服务：

```shell
systemctl daemon-reload
```

现在，我们可以用下面这组命令来操作redis了：

```shell
# 启动
systemctl start redis
# 停止
systemctl stop redis
# 重启
systemctl restart redis
# 查看状态
systemctl status redis
```

执行下面的命令，可以让redis开机自启：

```shell
systemctl enable redis
```









## 主从集群安装

主：具有读写操作

从：只有读操作

![image-20210630111505799](https://img2023.cnblogs.com/blog/2421736/202405/2421736-20240504151508059-1692865005.png)



1. 修改redis.conf文件

```properties
# 开启RDB
# save ""
save 3600 1
save 300 100
save 60 10000

# 关闭AOF
appendonly no
```

2. 将上面的redis.conf文件拷贝到不同地方

```shell
# 方式一：逐个拷贝
cp /usr/local/bin/redis-7.0.12/redis.conf /tmp/redis-7001
cp /usr/local/bin/redis-7.0.12/redis.conf /tmp/redis-7002
cp /usr/local/bin/redis-7.0.12/redis.conf /tmp/redis-7003

# 方式二：管道组合命令，一键拷贝
echo redis-7001 redis-7002 redis-7003 | xargs -t -n 1 cp /usr/local/bin/redis-7.0.12/redis.conf
```

4. 修改各自的端口、rdb目录改为自己的目录

```shell
sed -i -e 's/6379/7001/g' -e 's/dir .\//dir \/tmp\/redis-7001\//g' redis-7001/redis.conf
sed -i -e 's/6379/7002/g' -e 's/dir .\//dir \/tmp\/redis-7002\//g' redis-7002/redis.conf
sed -i -e 's/6379/7003/g' -e 's/dir .\//dir \/tmp\/redis-7003\//g' redis-7003/redis.conf
```

5. 修改每个redis节点的IP声明。虚拟机本身有多个IP，为了避免将来混乱，需要在redis.conf文件中指定每一个实例的绑定ip信息，格式如下：

```shell
# redis实例的声明 IP
replica-announce-ip IP地址


# 逐一执行
sed -i '1a replica-announce-ip 192.168.150.101' redis-7001/redis.conf
sed -i '1a replica-announce-ip 192.168.150.101' redis-7002/redis.conf
sed -i '1a replica-announce-ip 192.168.150.101' redis-7003/redis.conf

# 或者一键修改
printf '%s\n' redis-7001 redis-7002 redis-7003 | xargs -I{} -t sed -i '1a replica-announce-ip 192.168.150.101' {}/redis.conf
```

6. 启动

```shell
# 第1个
redis-server redis-7001/redis.conf
# 第2个
redis-server redis-7002/redis.conf
# 第3个
redis-server redis-7003/redis.conf


# 一键停止
printf '%s\n' redis-7001 redis-7002 redis-7003 | xargs -I{} -t redis-cli -p {} shutdown
```

7. 开启主从关系：配置主从可以使用replicaof 或者slaveof（5.0以前）命令

**永久配置**：在redis.conf中添加一行配置

```shell
slaveof <masterip> <masterport>
```

**临时配置**：使用redis-cli客户端连接到redis服务，执行slaveof命令（重启后失效）

```shell
# 5.0以后新增命令replicaof，与salveof效果一致
slaveof <masterip> <masterport>
```







# 卸载Redis

1. 查看redis是否启动

```shell
ps aux | grep redis
```

2. 若启动，则杀死进程

```shell
kill -9 PID
```

![image-20230724175911257](https://img2023.cnblogs.com/blog/2421736/202405/2421736-20240504151507577-1965722469.png)

3. 停止服务

```shell
redis-cli shutdown
```

4. 查看`/usr/local/lib`目录中是否有与Redis相关的文件

```shell
ll /usr/local/bin/redis-*

# 有的话就删掉

rm -rf /usr/local/bin/redis-*
```







# Nginx安装

- 安装gcc、pcrl库、zlib库

```bash
yum install -y gcc pcre pcre-devel zlib zlib-devel
```

- 加压tar包，进入解压后的目录，执行配置脚本

```bash
# 解压
tar -zxvf 下载的nginx压缩包

# 进入目录
cd nginx

# 执行配置脚本 --prefix是指定安装目录
./configure --prefix=/usr/local/nginx
```

- 进入解压目录执行编译安装

```bash
# 进入nginx解压后的目录，编译安装
make && make install
```





## Nginx启动

- 防火墙

```bash
# 关闭防火墙
systemctl stop firewalld.service

# 禁止防火墙开机自启
systemctl disable firewalld.service

# 放行指定端口的方式
firewall-cmd --zone=public --add-port=80/tcp --permanent

# 重置防火墙
firewall-cmd --reload
```



- 启动Nginx：进入安装好的目录 `/usr/local/nginx/sbin `

```bash
# 启动
./nginx
# 快速停止
./nginx -s stop
# 退出前完成已经接受的连接请求（不再接收新请求）
./nginx -s quit
# 重新加载配置
./nginx -s reload
```



通过系统服务启动：

- 编写服务脚本

```bash
vi /usr/lib/systemd/system/nginx.service
```

- 脚本内容：记得将下面路径改为自己的，前面通过脚本 `./configure --prefix=/usr/local/nginx` 安装在了指定目录

```bash
[Unit] 
Description=nginx - web server 
After=network.target remote-fs.target nss-lookup.target 

[Service] 
Type=forking 
PIDFile=/usr/local/nginx/logs/nginx.pid 
ExecStartPre=/usr/local/nginx/sbin/nginx -t -c /usr/local/nginx/conf/nginx.conf
ExecStart=/usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
ExecReload=/usr/local/nginx/sbin/nginx -s reload 
ExecStop=/usr/local/nginx/sbin/nginx -s stop 
ExecQuit=/usr/local/nginx/sbin/nginx -s quit 
PrivateTmp=true 

[Install] 
WantedBy=multi-user.target
```

脚本配置项解读：

```bash
[Unit]			控制部分，表示启动顺序和依赖关系
Description		简短描述
Documentation	文档地址
Requires	    当前 Unit 依赖的其他 Unit，如果它们没有运行，当前 Unit 会启动失败
Wants		    与当前 Unit 配合的其他 Unit，如果它们没有运行，当前 Unit 不会启动失败
BindsTo			与Requires类似，它指定的 Unit 如果退出，会导致当前 Unit 停止运行
Before			如果该字段指定的 Unit 也要启动，那么必须在当前 Unit 之后启动
After			如果该字段指定的 Unit 也要启动，那么必须在当前 Unit 之前启动
Conflicts		这里指定的 Unit 不能与当前 Unit 同时运行
Condition...	当前 Unit 运行必须满足的条件，否则不会运行
Assert...		当前 Unit 运行必须满足的条件，否则会报启动失败



[Service]		服务部分，表示服务的定义
Type			定义启动时的进程行为。它有以下几种值
                    Type=simple		默认值，执行ExecStart指定的命令，启动主进程
                    Type=forking	以 fork 方式从父进程创建子进程，创建后父进程会立即退出
                    Type=oneshot	一次性进程，Systemd 会等当前服务退出，再继续往下执行
                    Type=dbus		当前服务通过D-Bus启动
                    Type=notify		当前服务启动完毕，会通知Systemd，再继续往下执行
                    Type=idle		若有其他任务执行完毕，当前服务才会运行
ExecStart		启动当前服务的命令
ExecStartPre	启动当前服务之前执行的命令
ExecStartPost	启动当前服务之后执行的命令
ExecReload		重启当前服务时执行的命令
ExecStop		停止当前服务时执行的命令
ExecStopPost	停止当其服务之后执行的命令
RestartSec		自动重启当前服务间隔的秒数
Restart			定义何种情况 Systemd 会自动重启当前服务
					可能的值包括always（总是重启）、on-success、on-failure、on-abnormal、on-abort、on-watchdog
TimeoutSec		定义 Systemd 停止当前服务之前等待的秒数
Environment		指定环境变量



[Install]		安装部分，表示怎么进行安装配置
WantedBy		它的值是一个或多个 Target，当前 Unit 激活时（enable）符号链接会放入/etc/systemd/system目录下面以 Target 名 + .wants后缀构成的子目录中
RequiredBy		它的值是一个或多个 Target，当前 Unit 激活时，符号链接会放入/etc/systemd/system目录下面以 Target 名 + .required后缀构成的子目录中
Alias			当前 Unit 可用于启动的别名
Also			当前 Unit 激活（enable）时，会被同时激活的其他 Unit
DefaultInstance	 实例单元的限制，这个选项指定如果单元被允许运行默认的实例
WantedBy	     表示该服务所在的 Target
```



- 重新加载系统服务

```bash
systemctl daemon-reload
```

- 启动nginx服务

```bash
# 启动
systemctl start nginx.service

# 停止
systemctl stop nginx.service

# 查看状态
systemctl status nginx.service

# 重启
systemctl restart nginx.service

# 开机自启
systemctl enable nginx.service
```





# Tomcat安装

> 版本要和JDK对应，即JDK8对应Tomcat8

**1、准备安装包**：https://tomcat.apache.org/download-80.cgi

**2、上传，解压，重命名**：本人路径为`/usr/local`

```bash
# 解压
tar -zxvf apache-tomcat-.........tar.gz

# 重命名
mv pache-tomcat-8 tomcat8
```

**3、首次验证**：接进 `/usr/local/tomcat8` 目录，执行其中 bin 目录下的启动脚本

```bash
cd /usr/local/tomcat8/bin

./startup.sh
```

在浏览器中使用IP:8080能看到tomcat页面就行，要是看不到页面，那检查前面自己是否有配置错误

**4、配置快捷操作和开机自启**

```bash
cd /etc/rc.d/init.d/


touch tomcat


chmod +x tomcat


# 编辑上面的tomcat文件，内容如下，注意下面的路径
#!/bin/bash
# description: Tomcat7 Start Stop Restart
# processname: tomcat7
# chkconfig: 234 20 80
JAVA_HOME=/usr/local/jdk/jdk8
export JAVA_HOME
PATH=$JAVA_HOME/bin:$PATH
export PATH
CATALINA_HOME=/usr/local/tomcat8
export CATALINA_HOME
echo $CATALINA_HOME
case $1 in 
start)
sh $CATALINA_HOME/bin/startup.sh
echo 'tomcat8 start success'
;;
stop)
sh $CATALINA_HOME/bin/shutdown.sh
echo 'tomcat8 stop success'
;;
restart)
sh $CATALINA_HOME/bin/shutdown.sh
sh $CATALINA_HOME/bin/startup.sh
echo 'tomcat8 restart success'
;;
esac
echo 'end'
exit 0
```

- 后续就可以通过如下方式运行tomcat了

```bash
service tomcat start
service tomcat restart
service tomcat stop
```

- 加入开机自启

```bash
chkconfig --add tomcat

chkconfig tomcat on
```



# Docker安装

**1、安装**

```bash
yum install -y docker

# 查看安装结果
docker version
```

**2、开启服务**

```bash
systemctl start docker.service
```

**3、开启自启**

```bash
systemctl enable docker.service
```

**4、更新本地镜像源为阿里镜像源**

- 参考阿里云的镜像加速文档：https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors

```shell
vim /etc/docker/daemon.json


{
 "registry-mirrors": ["https://838ztoaf.mirror.aliyuncs.com"]
}
```

**5、重新加载配置、重启服务**

```bash
systemctl daemon-reload

systemctl restart docker.service
```



## 修订：2024年7月份后幺蛾子出了

2024年7月份，很多东西要么停止维护，要么直接整改，如上面安装docker使用阿里云镜像仓库个人版也没什么卵用时，使用 `docker pull` 不顶用时，解决方式直接 [去这里](https://blog.csdn.net/weixin_46028606/article/details/142663559) 体验别人RMB的力量

> 修改后 `docker pull` 还出现拉取不下来的，直接pull指定镜像仓库地址，如 `docker pull daocloud.io/library/redis:latest`

再提供一个Docker国内镜像源：

```bash
vim /etc/docker/daemon.json 

#-------------------------------
{
"registry-mirrors":["https://docker.rainbond.cc"]
}
#-------------------------------

# 重新加载配置
sudo systemctl daemon-reload

# 重启docker
sudo systemctl restart docker
```



## Docker基本命令

> 在前面配置了docker镜像后，`docker pull`时失败借鉴：[docker pull timeout](https://blog.csdn.net/m0_63289178/article/details/135202951)

Docker仓库地址(即dockerHub)：https://hub.docker.com
![](https://img2023.cnblogs.com/blog/2421736/202405/2421736-20240504140629131-1038375760.png)


常见的镜像操作命令如图：

![image-20210731155649535](https://img2023.cnblogs.com/blog/2421736/202405/2421736-20240504140629434-587681271.png)



```shell
# 拉取镜像
docker pull 镜像名称

# 查看全部镜像
docker images

# 删除镜像
docker rmi 镜像ID

# 将本地的镜像导出 
docker save -o 导出的路径 镜像id

# 加载本地的镜像文件 
docker load -i 镜像文件

# 修改镜像名称 
docker tag 镜像id 新镜像名称:版本




# 简单运行操作 
docker run 镜像ID | 镜像名称
# docker run	指的是创建一个容器并运行

# 跟参数的运行
docker run -d -p 宿主机端口:容器端口 --name 容器名称 镜像ID | 镜像名称
# 如：docker run -d -p 8081:8080 --name tomcat b8
# -d：代表后台运行容器 
# -p 宿主机端口:容器端口		为了映射当前Linux的端口和容器的端口 
# --name 容器名称：指定容器的名称

# 查看运行的容器
docker ps [-qa]
# -a：查看全部的容器，包括没有运行 
# -q：只查看容器的标识

# 查看日志
docker logs -f 容器id 
# -f：可以滚动查看日志的最后几行

# 进入容器内部
docker exec -it 容器id bash 
# docker exec 进入容器内部，执行一个命令
# -it	给当前进入的容器创建一个标准输入、输出终端，允许我们与容器交互
# bash	进入容器后执行的命令，bash是一个Linux终端交互命令
# 退出容器：exit

# 将宿主机的文件复制到容器内部的指定目录
docker cp 文件名称 容器id:容器内部路径 
docker cp index.html 982:/usr/local/tomcat/webapps/ROOT

# 重新启动容器
docker restart 容器id

# 启动停止运行的容器
docker start 容器id

# 停止指定的容器（删除容器前，需要先停止容器）
docker stop 容器id

# 停止全部容器
docker stop $(docker ps -qa)

# 删除指定容器
docker rm 容器id

# 删除全部容器
docker rm $(docker ps -qa)




# ==================数据卷volume========================

# 创建数据卷
docker volume create 数据卷名称
# 创建数据卷之后，默认会存放在一个目录下 /var/lib/docker/volumes/数据卷名称/_data

# 查看数据卷详情
docker volume inspect 数据卷名称

# 查看全部数据卷
docker volume ls

# 删除指定数据卷
docker volume rm 数据卷名称



# Docker容器映射数据卷==========>有两种方式：
# 1、通过数据卷名称映射，如果数据卷不存在。Docker会帮你自动创建，会将容器内部自带的文件，存储在默认的存放路径中

# 通过数据卷名称映射
docker run -v 数据卷名称:容器内部的路径 镜像id

# 2、通过路径映射数据卷，直接指定一个路径作为数据卷的存放位置。但是这个路径不能是空的 - 重点掌握的一种
# 通过路径映射数据卷 
docker run -v 路径:容器内部的路径 镜像id

# 如：docker run -d -p 8081:8080 --name tomcat -v[volume] /opt/tocmat:/usr/local/tocmat/webapps b8
```

数据卷挂载和目录直接挂载的区别：
1. 数据卷挂载耦合度低，由docker来管理目录且目录较深，所以不好找
2. 目录挂载耦合度高，需要我们自己管理目录，不过很容易查看


> 更多命令通过 `docker -help` 或 `docker 某指令 --help` 来学习






# Docker-Compose

Docker Compose可以基于Compose文件帮我们快速的部署分布式应用，而无需手动一个个创建和运行容器！





## 安装Docker-Compose

**1、下载Docker-Compose**

```shell
# 1、安装
# 1.1、选择在线，直接官网拉取
curl -L https://github.com/docker/compose/releases/download/1.23.1/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose

# 要是嫌慢的话，也可以去这个网址
curl -L https://get.daocloud.io/docker/compose/releases/download/1.26.2/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose

# 1.2、也可以选择离线安装，直接下载到本地后，上传到虚拟机 /usr/local/bin/ 路径中即可



# 2、修改文件权限,因为 /usr/local/bin/docker-compose 文件还没有执行权
chmod +x /usr/local/bin/docker-compose

# 3、检测是否成功，出现命令文档说明就表示成功了
docker-compose
```



可以再加上一个东西：Base自动补全命令

```shell
# 补全命令
curl -L https://raw.githubusercontent.com/docker/compose/1.29.1/contrib/completion/bash/docker-compose > /etc/bash_completion.d/docker-compose

# 若是出现错误，这是因为上面这个网址域名的问题，这需要修改hosts文件
# 可以先修改hosts，然后再拉取Base自动补全命令
echo "199.232.68.133 raw.githubusercontent.com" >> /etc/hosts
```

附：上面提到的DaoCloud镜像市场去这里：https://hub.daocloud.io/


## docker-compose的基本命令

> 在使用docker-compose的命令时，默认会在当前目录下找docker-compose.yml文件，所以：需要让自己在创建的docker-compose.yml文件的当前目录中




```shell
# 1. 基于docker-compose.yml启动管理的容器
docker-compose up -d

# 2. 关闭并删除容器
docker-compose down

# 3. 开启|关闭|重启已经存在的由docker-compose维护的容器
docker-compose start|stop|restart

# 4. 查看由docker-compose管理的容器
docker-compose ps

# 5. 查看日志
docker-compose logs -f [服务名1] [服务名2]
```



# RabbitMQ安装

查看自己的Linux版本

```shell
uname -a
```

![image](https://img2023.cnblogs.com/blog/2421736/202405/2421736-20240504162903638-75699466.png)



**1、下载erlang，上传到服务器**：因RabbitMQ是基于这玩意儿写的，本人上传路径 `/usr/local/rabbitmq`

下载地址：https://github.com/rabbitmq/erlang-rpm/releases ， 选择自己需要的版本即可，本文选择的是 erlang-23.3.4.8-1.el7.x86_64.rpm

> RabbitMQ和Erlang的版本对应关系链接地址 https://www.rabbitmq.com/which-erlang.html

注意一个问题：要看是基于什么Linux的版本

![image-20240504163107914](https://img2023.cnblogs.com/blog/2421736/202405/2421736-20240504162943472-293665137.png)



**2、安装erlang**

```bash
rpm -ivh erlang-23.3.4.8-1.el7.x86_64.rpm
```

> **提示**
>
> 需要保证自己的Linux中有rpm命令，没有的话，执行`yum install rpm`指令即可安装rpm



**3、安装Rabbitmq需要的依赖**

```bash
yum install socat -y
```



**4、下载RabbitMQ，上传到服务器，安装**：本人上传地址`/usr/local/rabbitmq`

GitHub下地址https://github.com/rabbitmq/rabbitmq-server/releases ， 选择自己要的版本即可，本文选择的是 rabbitmq-server-3.9.15-1l7.noarch.rpm

```bash
rpm -ivh rabbitmq-server-3.9.15-1.el7.noarch.rpm
```

**5、启动RabbitMQ**

```bash
# 启动服务
/sbin/service rabbitmq-server start

# 停止服务
/sbin/service rabbitmq-server stop

# 查看启动状态
/sbin/service rabbitmq-server status

# 开启开机自启
chkconfig rabbitmq-server on
```

**6、安装web管理插件**

```bash
# 1、停止RabbitMQ服务
service rabbitmq-server stop   # 使用上面5中的命令 /sbin/service rabbitmq-server stop也行

# 2、安装插件
rabbitmq-plugins enable rabbitmq_management

# 3、开启RabbitMQ服务
service rabbitmq-server start
```

目录浏览器访问 ip:15672 还不行，需要创建用户

> 目前默认用户名/密码：guest，权限是administrator，所以可以直接执行下面指令的最后一步

```bash
# 查看当前用户 / 角色有哪些
rabbitmqctl list_users

# 删除用户
rabbitmqctl delete_user 用户名

# 添加用户
rabbitmqctl add_user 用户名 密码

# 设置用户角色
rabbitmqctl set_user_tags 用户名 administrator

# 设置用户权限	ps：guest角色就是没有这一步
rabbitmqctl set_permissions -p "/" 用户名 ".*" ".*" ".*"
# 设置用户权限指令解释
			set_permissions [-p <vhostpath>] <user> <conf> <write> <read>
```

现在就可以使用 ip:15672 访问了

要是访问不了，注意防火墙的事

```bash
# 查看防火墙状态
systemctl status firewalld

# 关闭防火墙
systemctl stop firewalld

# 一劳永逸 禁用防火墙
systemctl disable firewalld

# ============================================

# 当然：上面的方式不建议用，可以用如下的方式

# 6379端口号是否开放
firewall-cmd --query-port=6379/tcp

# 开放6379端口
firewall-cmd --permanent --add-port=6379/tcp

#重启防火墙(修改配置后要重启防火墙)
firewall-cmd --reload
```







## 使用Docker安装

**1、查看自己的docker容器中是否已有了rabbitmq这个名字的镜像**

```shell
docker images

# 有的话就删除镜像
docker rmi 镜像ID
```




**2、拉取RabbitMQ镜像 并 启动Docker容器**

```shell
# 5672:5672 是消息之间通讯的端口  15672:15672 是管理界面的端口
# \ 表示命令拼接，就是换行，接着读取命令
# -e RABBITMQ_DEFAULT_USER=zixieqing 直接设置了登录的用户名，下一个同理，设置登录密码
docker run -it --rm --name rabbitmq \
 -e RABBITMQ_DEFAULT_USER=zixieqing \
 -e RABBITMQ_DEFAULT_PASS=072413 \
 -p 5672:5672 -p 15672:15672 \
 --hostname mq1 \
 -d \
 rabbitmq:3.9-management
```



**3、查看Docker容器是否启动**

```shell
docker ps
```


再次在浏览器进行访问，使用设置的用户名和密码就可以吃鸡了







# Python安装

系统自带了一个Python 2.7.5

![image-20240504174019534](https://img2023.cnblogs.com/blog/2421736/202405/2421736-20240504173855930-2118591284.png)

退出上述交互界面指令：

```bash
exit()	或 ctrl + d
```

现在主流都是 Python3 ，所以接下来再装⼀个 Python3 ，打造⼀个共存的环境



**1、准备安装包**：[Python官网下载](https://www.python.org/downloads/source/)，本文选择的是 Python-3.10.14.tgz

**2、上传，解压**：本人上传路径 `/root`

```bash
tar zxvf Python-3.10.14.tgz
```

**3、安装依赖**

```bash
yum install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlitedevel readline-devel tk-devel gcc make
```

**4、编译安装**

```bash
# 进入解压目录
cd /root/Python-3.10.14

# prefix 改为自己的
./configure prefix=/usr/local/python3

# 编译安装
make && make install
```

**5、添加软连接**：将刚刚安装⽣成的⽬录 /usr/local/python3 ⾥的 python3 可执⾏⽂件做⼀份软链接，链接到 /usr/bin 下，⽅便后续使⽤python3

```bash
ln -s /usr/local/python3/bin/python3 /usr/bin/python3

ln -s /usr/local/python3/bin/pip3 /usr/bin/pip3
```

**6、验证**：输入`python3 `，即可查看 Python3 版本的安装结果

而输入 `python`，依然还是 python 2.7.5 环境，实现了共存

![image-20240504175348978](https://img2023.cnblogs.com/blog/2421736/202405/2421736-20240504175224695-931580506.png)





# NodeJS安装

**1、下载安装包**：官网地址 https://nodejs.org/dist/ 本文选择的是 node-v16.20.2-linux-x64.tar.gz 

**2、上传，解压，重命名**：本人上传路径 `/usr/local`

```bash
# 解压
tar -zxvf node-v16.20.2-linux-x64.tar.gz

# 重命名
mv node-v16.20.2-linux-x64 node16
```

**3、配置环境变量**

```bash
# 编辑文件
vim ~/.bash_profile


# 在文件尾部加入如下内容	路径改为自己的
# Nodejs
export PATH=/usr/local/node16/bin:$PATH
```

**4、让环境生效**

```bash
source ~/.bash_profile
```

**5、验证**

```bash
node -v

npm version

npx -v
```











