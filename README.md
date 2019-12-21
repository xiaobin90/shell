# shell
shell脚本整理
1.
#!/bin/bash
#创建用户及密码
useradd "$1"
echo "$2" | passwd --stdin "$1"

2.编写脚本vim /root/logbak.sh
#!/bin/bash
#每周五3:00备份/var/log日志文件,备份文件应该加上日期,避免后续日志被覆盖
tar -czvPf log-`date+%Y%m%d`.tar.gz /var/log

#编写周期性计划任务执行上述脚本
crontab -e 
00 03 * * 5 /root/logbak.sh

3.一键部署LNMP
#!/bin/bash
#使用yum一键配置LNMP架构
yum -y install httpd
yum -y insstall mariadb-server mariadb mariadb-devel
yum -y install php php-mysql
systemctl start httpd
systemctl start mariadb
systemctl enable httpd
systemctl enable mariadb

4.源码一键部署LNMP
#! /bin/bash
# 一键部署安装多台lnmp
#在一台服务器上生成秘钥,做无密码认证
ssh-keygen -f /root/.ssh/id_rsa -N ''
for i in {1..10}
do
  ssh-copy-id 192.168.4.$i
done

5.随机生成8位密码
#! /bin/bash
a=abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789  #定义变量a为52个大小写字母和0-9数字
for i in {1..8}    
do
  x=$[RANDOM%62]     #定义变量x取值从62%62起始位开始
  p=${a:x:1}         #从定义的变量a中的x位置截取一位
  pa=$pa$p           #将每次循环截取的取值组合在一起
done
echo $pa           #输出随机截取的8位密码

6.批量修改扩展名
#!/bin/bash
for i in `ls*.txt`
do
  x=${i%.*}       #截取没有文件扩展名的文件名
  mv $i $x.doc    #改名时将这个文件名与.doc组合
done
7.vim k.sh批量将.doc修改回.txt
#!/bin/bash
for i in `ls *.$1`
do
  x=${i%.$1}
  mv $i $x.$2
done
chmod +x k.sh
bash k.sh doc txt

7.一键部署zabbix监控
#!/bin/bash
#一键部署服务器端zabbix监控软件
#需要提前配置好yum源,将zabbix,nginx压缩包拷贝至/root/下
#
#
#
#第一步,在监控服务器上部署搭建lnmp架构
yum -y install gcc pcre-devel openssl-devel zlib-devel
tar -xzvf nginx-1.12.2.tar.gz
cd nginx-1.12.2/
./configure --with-http_ssl_module
make&&make install
yum -y install php php-mysql php-fpm mariadb mariadb-server mariadb-devel
sed -i '65,68s/#//' /usr/local/nginx/conf/nginx.conf
sed -i '70,71s/#//' /usr/local/nginx/conf/nginx.conf
sed -i '70s/fastcgi_params;/fastcgi.conf;/' /usr/local/nginx/conf/nginx.conf
systemctl start mariadb
systemctl php-fpm
/usr/local/nginx/sbin/nginx
echo '<?php  $i="hello world!!"; echo $i; ?>' > /usr/local/nginx/html/test.php
curl http://192.168.2.17/test.php
#
#
#
#第二部安装zabbix软件
#安装zabbix软件
yum -y install net-snmp-devel curl-devel libevent-devel  #安装软件依赖包
cd
tar -xf zabbix-3.4.4.tar.gz  #解压zabbix压缩包
cd /root/zabbix-3.4.4/   #安装所需模块
./configure --enable-server --enable-proxy --enable-agent --with-mysql=/usr/bin/mysql_config --with-net-snmp --with-libcurl
make install #编译安装
#
#
#
#第三部设置数据库
mysqladmin -uroot password 'zabbix'
mysql -uroot -pzabbix -e "create database zabbix character set utf8;"
mysql -uroot -pzabbix -e "grant all on zabbix.* to zabbix@'localhost' identified by 'zabbix';"
mysql -uzabbix -pzabbix zabbix < /root/zabbix-3.4.4/database/mysql/schema.sql
mysql -uzabbix -pzabbix zabbix < /root/zabbix-3.4.4/database/mysql/images.sql
mysql -uzabbix -pzabbix zabbix < /root/zabbix-3.4.4/database/mysql/data.sql
cd /root/zabbix-3.4.4/frontends/php/
cp -a * /usr/local/nginx/html/
chmod -R 777 /usr/local/nginx/html/*
sed -i '17s/http {/http { \t\n/' /usr/local/nginx/conf/nginx.conf
sed -i '18s/^$/\t\nfastcgi_buffers 8 16k;\n\tfastcgi_buffer_size 32k;\n\tfastcgi_connect_timeout 300;\n\tfastcgi_send_timeout 300;\n\tfastcgi_read_timeout 300;\n/'  /usr/local/nginx/conf/nginx.conf  #修改nginx配置文件,以满足zabbix运行参数
/usr/local/nginx/sbin/nginx -s stop #停止nginx加载新添配置
/usr/local/nginx/sbin/nginx
#
#
#
#第四部安装php依赖软件
cd /root/
yum -y install php-gd php-xml php-ldap
yum -y install php-bcmath php-mbstring
sed -i '878s/;date.timezone = */date.timezone = Asia Shanghai/' /etc/php.ini
sed -i '384s/max_execution_time = 30/max_execution_time = 300/' /etc/php.ini
sed -i '672s/post_max_size = 8M/post_max_size = 32M/' /etc/php.ini
sed -i '394s/max_input_time = 60/max_input_time = 300/' /etc/php.ini
systemctl restart php-fpm
#
#
#
#测试
echo "测试lnmp"
curl http://192.168.2.17/test.php
#查看zabbix目录
echo "查看zabbix目录文件"
ls /usr/local/etc
ls /usr/local/bin
ls /usr/local/sbin



~                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           
~                                       
