安装fail2ban服务
```
apt install fail2ban
```
启动、设置开机自启动、查看状态
```
systemctl start fail2ban
systemctl enable fail2ban
systemctl status fail2ban
``` 
配置
[ ! ] 这里不建议直接修改/etc/fail2ban/jail.conf文件里面的参数
可以创建/etc/fail2ban/jail.local文件它会默认覆盖掉jail.conf文件里面的参数
``` 
vim /etc/fail2ban/jail.local
``` 
设置以下参数
```
[sshd]               
enabled = true       
port = 22 #默认端口号如不是请修改           
filter = sshd      
logpath = /var/log/fail2ban.log  #日志
maxretry = 3        #最大尝试次数
findtime = 8467200 #
bantime = 8467200 # 解用时间
backend = systemd
ignoreip = 127.0.0.1/8 192.168.0.1/16  #白名单网段/地址
```
重新启动Fail2ban服务
```
systemctl restart fail2ban
```
查看fail2ban ssh进程的禁用列表
 ```
fail2ban-client status sshd
```
解禁ban列表内IP
```
fail2ban-client set sshd unbanip [IP address]
```
>例如：fail2ban-client set sshd unbanip 8.245.78.2
表示解禁8.245.78.2这个IP地址

卸载服务
```
systemctl stop fail2ban 
apt purge fail2ban
```