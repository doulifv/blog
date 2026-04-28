### 启用 IP 转发 (Kernel Level)
#### 1.编辑 sysctl 配置文件
```
vi /etc/sysctl.conf
```
#### 2.取消注释或添加以下行

- 对于 IPv4：
```
net.ipv4.ip_forward=1
```
- 对于 IPv6 (如果需要)：
```
net.ipv6.conf.all.forwarding=1
```
#### 3.应用更改
```
sudo sysctl -p
```