查看需要配置的网卡名
```
ip address
```
编辑网卡配置文件/etc/network/interfaces
```
vi /etc/network/interfaces
```
添加以下字段为ens33这张网卡添加192.168.1.2这IP地址
```
auto ens33
iface ens33 inet static
address 192.168.1.2/24 #主机地址/掩码
gateway 192.168.1.1 #网关
```
添加DNS服务器地址
```
echo >> /etc/resolv.conf  nameserver 8.8.8.8
echo >> /etc/resolv.conf  nameserver 1.1.1.1
```
重启网卡、查看网卡IP配置情况
```
/etc/init.d/networking restart
ip address
```
允许root用户远程登录、重启ssh服务
```
echo PermitRootLogin yes >> /etc/ssh/sshd_config  
/etc/init.d/ssh restart
```