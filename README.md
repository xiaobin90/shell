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




