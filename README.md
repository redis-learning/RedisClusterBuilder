# RedisClusterBuilder

how to build it

搭建的机器为以下6台机器
192.168.102.132 7001
192.168.102.132 7002
192.168.102.130 7003
192.168.102.130 7004
192.168.102.131 7005
192.168.102.131 7006

测试添加主节点和备节点（扩容和缩容）

192.168.102.133 7007
192.168.102.133 7008


OS： cenos7


1.修改IP
/etc/sysconfig/network-scripts/

2.关闭防火墙
systemctl stop firewalld.service
systemctl disable firewalld.service

ping 的通

4.下载wget和gcc-c++
yum -y install wget
yum -y install gcc-c++

5.下载最新版本的Redis(redis.io 好像是4.0+)
具体可以参考解压包下面的readme.md
make MALLOC=libc
make && make install

6.copy所有的脚本在src下的到/usr/local/redis/etc/
主要是redis server. redis client. redis benchmark. redis-trib
在util的包下面有一个启动脚本可以用/usr/local/redis-4.0.9/utils/redis_init_script

可以放在/etc/init.d并设置ckgconfig来设置开机启动。具体可以自行baidu


下面是集群的搭建

配置文件的修改
bind 192.168.102.133
port 7007
daemonize yes
pidfile /usr/local/redis-cluster/7007/redis_7007.pid
logfile "/usr/local/redis-cluster/7007/redis-7007.log"
dir /usr/local/redis-cluster/7007
appendonly yes
cluster-enabled yes
cluster-config-file nodes-7007.conf
cluster-node-timeout 5000

redis_init_script  copy from util package
REDISPORT=7007
EXEC=/usr/local/redis/etc/redis-server
CLIEXEC=/usr/local/redis/etc/redis-cli

PIDFILE=/usr/local/redis-cluster/${REDISPORT}/redis_${REDISPORT}.pid
CONF="/usr/local/redis-cluster/${REDISPORT}/${REDISPORT}.conf"



在执行gem install redis
	gem install redis
    ERROR:  Error installing redis:
            redis requires Ruby version >= 2.2.2.
			
	解决办法
	gpg2 --keyserver hkp://keys.gnupg.net --recv-keys D39DC0E3
    curl -L get.rvm.io | bash -s stable
	source /usr/local/rvm/scripts/rvm
	rvm list known
	rvm install 2.3.3
	rvm use 2.3.3
	rvm use 2.3.3 --default
	gem install redis

./redis-trib.rb create --replicas 1 192.168.102.132:7001 192.168.102.132:7002 192.168.102.130:7003 192.168.102.130:7004 192.168.102.131:7005 192.168.102.131:7006
	
	
M: c7e499e2797fdf9a4a493edde175ebdaeaa70879 192.168.102.132:7001
   slots:0-5460 (5461 slots) master
   1 additional replica(s)
S: 9a299581151b01b768a59531190c1812c375943f 192.168.102.131:7006
   slots: (0 slots) slave
   replicates 7869895e507894d71077d23b405c6b8ea4806ac5
M: b26ae2c5eb1301f1b52aec918c38b99a66ef6c19 192.168.102.131:7005
   slots:10923-16383 (5461 slots) master
   1 additional replica(s)
S: c6ff141ac5532314c8a31ea885bf298437cb793d 192.168.102.130:7004
   slots: (0 slots) slave
   replicates c7e499e2797fdf9a4a493edde175ebdaeaa70879
M: 7869895e507894d71077d23b405c6b8ea4806ac5 192.168.102.130:7003
   slots:5461-10922 (5462 slots) master
   1 additional replica(s)
S: ee38b0a9bb4b88dcd709ceaefe33dcb4d845c9f6 192.168.102.132:7002
   slots: (0 slots) slave
   replicates b26ae2c5eb1301f1b52aec918c38b99a66ef6c19
   
添加主节点
 ./redis-trib.rb add-node 192.168.102.133:7007 192.168.102.132:7001
 
 [root@host1 etc]# ./redis-trib.rb check 192.168.102.132:7001
>>> Performing Cluster Check (using node 192.168.102.132:7001)
M: c7e499e2797fdf9a4a493edde175ebdaeaa70879 192.168.102.132:7001
   slots:0-5460 (5461 slots) master
   1 additional replica(s)
S: 9a299581151b01b768a59531190c1812c375943f 192.168.102.131:7006
   slots: (0 slots) slave
   replicates 7869895e507894d71077d23b405c6b8ea4806ac5
M: b26ae2c5eb1301f1b52aec918c38b99a66ef6c19 192.168.102.131:7005
   slots:10923-16383 (5461 slots) master
   1 additional replica(s)
M: ba08aada042219a5976bbc7b65b2fd2426dd2d7b 192.168.102.133:7007
   slots: (0 slots) master
   0 additional replica(s)
S: c6ff141ac5532314c8a31ea885bf298437cb793d 192.168.102.130:7004
   slots: (0 slots) slave
   replicates c7e499e2797fdf9a4a493edde175ebdaeaa70879
M: 7869895e507894d71077d23b405c6b8ea4806ac5 192.168.102.130:7003
   slots:5461-10922 (5462 slots) master
   1 additional replica(s)
S: ee38b0a9bb4b88dcd709ceaefe33dcb4d845c9f6 192.168.102.132:7002
   slots: (0 slots) slave
   replicates b26ae2c5eb1301f1b52aec918c38b99a66ef6c19
   
move 4096到新的节点
./redis-trib.rb reshard 192.168.102.132:7001
M: c7e499e2797fdf9a4a493edde175ebdaeaa70879 192.168.102.132:7001
   slots:1365-5460 (4096 slots) master
   1 additional replica(s)
S: 9a299581151b01b768a59531190c1812c375943f 192.168.102.131:7006
   slots: (0 slots) slave
   replicates 7869895e507894d71077d23b405c6b8ea4806ac5
M: b26ae2c5eb1301f1b52aec918c38b99a66ef6c19 192.168.102.131:7005
   slots:12288-16383 (4096 slots) master
   1 additional replica(s)
M: ba08aada042219a5976bbc7b65b2fd2426dd2d7b 192.168.102.133:7007
   slots:0-1364,5461-6826,10923-12287 (4096 slots) master
   0 additional replica(s)
S: c6ff141ac5532314c8a31ea885bf298437cb793d 192.168.102.130:7004
   slots: (0 slots) slave
   replicates c7e499e2797fdf9a4a493edde175ebdaeaa70879
M: 7869895e507894d71077d23b405c6b8ea4806ac5 192.168.102.130:7003
   slots:6827-10922 (4096 slots) master
   1 additional replica(s)
S: ee38b0a9bb4b88dcd709ceaefe33dcb4d845c9f6 192.168.102.132:7002
   slots: (0 slots) slave
   replicates b26ae2c5eb1301f1b52aec918c38b99a66ef6c19

添加一个新的备节点给c7e499e2797fdf9a4a493edde175ebdaeaa70879， 所以这个主节点就有2个备节点

./redis-trib.rb add-node --slave --master-id c7e499e2797fdf9a4a493edde175ebdaeaa70879 192.168.102.133:7008 192.168.102.132:7001
修改从节点
[root@localhost etc]# ./redis-cli -c -h 192.168.102.133 -p 7008 cluster replicate b26ae2c5eb1301f1b52aec918c38b99a66ef6c19
OK
M: c7e499e2797fdf9a4a493edde175ebdaeaa70879 192.168.102.132:7001
   slots:1365-5460 (4096 slots) master
   1 additional replica(s)
S: 9a299581151b01b768a59531190c1812c375943f 192.168.102.131:7006
   slots: (0 slots) slave
   replicates 7869895e507894d71077d23b405c6b8ea4806ac5
M: b26ae2c5eb1301f1b52aec918c38b99a66ef6c19 192.168.102.131:7005
   slots:12288-16383 (4096 slots) master
   2 additional replica(s)
M: ba08aada042219a5976bbc7b65b2fd2426dd2d7b 192.168.102.133:7007
   slots:0-1364,5461-6826,10923-12287 (4096 slots) master
   0 additional replica(s)
S: c6ff141ac5532314c8a31ea885bf298437cb793d 192.168.102.130:7004
   slots: (0 slots) slave
   replicates c7e499e2797fdf9a4a493edde175ebdaeaa70879
M: 7869895e507894d71077d23b405c6b8ea4806ac5 192.168.102.130:7003
   slots:6827-10922 (4096 slots) master
   1 additional replica(s)
S: ee38b0a9bb4b88dcd709ceaefe33dcb4d845c9f6 192.168.102.132:7002
   slots: (0 slots) slave
   replicates b26ae2c5eb1301f1b52aec918c38b99a66ef6c19
S: 5be1f81c082a7d409d3ac6b940a04b3ef7ceeae9 192.168.102.133:7008
   slots: (0 slots) slave
   replicates b26ae2c5eb1301f1b52aec918c38b99a66ef6c19
   
 删除主节点， 把hashslot都分配给其他的master
分三次reshard给不同的的master

[root@host1 etc]# ./redis-trib.rb del-node 192.168.102.132:7001 ba08aada042219a5976bbc7b65b2fd2426dd2d7b


关闭其中一个只有一个slave的节点，拥有两个的自动会分配一个给他
M: c7e499e2797fdf9a4a493edde175ebdaeaa70879 192.168.102.132:7001
   slots:0-5460 (5461 slots) master
   1 additional replica(s)

shutdown
S: c6ff141ac5532314c8a31ea885bf298437cb793d 192.168.102.130:7004
   slots: (0 slots) slave
   replicates c7e499e2797fdf9a4a493edde175ebdaeaa70879

自动切换完毕
M: c7e499e2797fdf9a4a493edde175ebdaeaa70879 192.168.102.132:7001
   slots:0-5460 (5461 slots) master
   1 additional replica(s)
S: 9a299581151b01b768a59531190c1812c375943f 192.168.102.131:7006
   slots: (0 slots) slave
   replicates 7869895e507894d71077d23b405c6b8ea4806ac5
M: b26ae2c5eb1301f1b52aec918c38b99a66ef6c19 192.168.102.131:7005
   slots:5461-6825,12288-16383 (5461 slots) master
   1 additional replica(s)
M: 7869895e507894d71077d23b405c6b8ea4806ac5 192.168.102.130:7003
   slots:6826-12287 (5462 slots) master
   1 additional replica(s)
S: ee38b0a9bb4b88dcd709ceaefe33dcb4d845c9f6 192.168.102.132:7002
   slots: (0 slots) slave
   replicates b26ae2c5eb1301f1b52aec918c38b99a66ef6c19
S: 5be1f81c082a7d409d3ac6b940a04b3ef7ceeae9 192.168.102.133:7008
   slots: (0 slots) slave
   replicates c7e499e2797fdf9a4a493edde175ebdaeaa70879  