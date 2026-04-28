首先安装需要用到的依赖
```
apt install wget gnupg lsb-release -y
```
使用wget获取deb包、安装  | 需要其他版本可前往[MySQL官网下载](https://dev.mysql.com/downloads/)
```
wget https://dev.mysql.com/get/mysql-apt-config_0.8.28-1_all.deb
dpkg -i mysql-apt-config_0.8.28-1_all.deb 
```
![选择OK即可](https://upload-images.jianshu.io/upload_images/27021437-9f963c9264a73d55.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
更新软件包、安装mysql-server
```
apt update
apt install mysql-server
```
![设置一个可靠的密码](https://upload-images.jianshu.io/upload_images/27021437-2cd8bc2131ef6f72.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
至此MySQL服务已经部署完毕。
___
启动服务、设置开机启动、查看进程状态
```
systemctl start mysql
systemctl enable mysql
systemctl status mysql
```
查看MySQL版本
```
mysql -V
```
进入MySQL
```
mysql -u root -p
#Enter password:
#退出数据库 
#mysql> quit;
#Bye
```
查看日志
```
journalctl -u mysql -xe
tail -f /var/log/mysql/mysqld.log
```