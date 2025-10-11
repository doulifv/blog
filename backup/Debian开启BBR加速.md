Debian10-12默认编译了BBR拥塞控制算法可直接开启
命令如下
```
echo net.core.default_qdisc=fq >> /etc/sysctl.conf
echo net.ipv4.tcp_congestion_control=bbr >> /etc/sysctl.conf
sysctl -p
```
检查是否配置成功
```
sysctl net.ipv4.tcp_available_congestion_control
```
回显如下信息则表示成功开启BBR加速
>net.ipv4.tcp_available_congestion_control = reno cubic bbr
