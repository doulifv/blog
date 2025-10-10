#一，安装需要的依赖
```
apt install wireguard
```
开启流量转发
```
echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf
sysctl -p
```
创建Wireguard && 设置777任意读取执行权限
```
mkdir -p /etc/wireguard && chmod 0777 /etc/wireguard
cd /etc/wireguard
```
生成公、私钥文件
```
wg genkey | tee server_privatekey | wg pubkey > server_publickey
wg genkey | tee client_privatekey | wg pubkey > client_publickey
```
生成的配置文件路径：/etc/wireguard/wg0.conf，命令如下：
```
echo "
[Interface]
PrivateKey = $(cat server_privatekey) # 填写本机的privatekey 内容
Address = 10.0.8.1/24
PostUp   = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -A FORWARD -o wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -D FORWARD -o wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
#注意修改对应网卡名称
ListenPort = 50814 # 注意该端口是UDP端口
DNS = 8.8.8.8
MTU = 1420
[Peer]
PublicKey =  $(cat client_publickey)  # 填写对端的publickey 内容
AllowedIPs = 10.0.8.10/24 " > wg0.conf
```
还需要设置开机启动：
```
systemctl enable wg-quick@wg0
```
#客户端配置文件生成
```echo "
[Interface]
  PrivateKey = $(cat client_privatekey)  # 填写本机的privatekey 内容
  Address = 10.0.8.10/24
  DNS = 8.8.8.8
  MTU = 1420

[Peer]
  PublicKey = $(cat server_publickey)  # 填写对端的publickey 内容
  Endpoint = server公网的IP:50814
  AllowedIPs = 0.0.0.0/0, ::0/0
  PersistentKeepalive = 25 " > client.conf
```
# 2.2.4 步骤四: 启动 | 停止

## 2.2.4.1 服务端启动

启动或停止**wireguard**服务端的命令如下：
 启动WireGuard
```
wg-quick up wg0
```
停止WireGuard
```
wg-quick down wg0
```
查看wireguard服务端运行状态命令：
```
wg
```
##2.2.4.2 客户端启动
启动或停止wireguard客户端的命令如下：
启动WireGuard
```
wg-quick up client
```
停止WireGuard
```
wg-quick down client
```
---
[Err-1](https://askubuntu.com/questions/1456439/wireguard-error-message-resolvconf-command-not-found)
/usr/bin/wg-quick: line 32: resolvconf: command not found
```
apt install openresolv
```
[Err-2]
/usr/bin/wg-quick: line 295: iptables: command not found
```
apt install iptables
```
---
参考教程
[Installation - WireGuard](https://www.wireguard.com/install/)
[组网神器WireGuard安装与配置教程（超详细）_wirguard-CSDN博客](https://blog.csdn.net/qq_20042935/article/details/127089626)
[/usr/bin/wg-quick：第 31 行：resolvconf：找不到命令 [WireGuard | Debian] - 超级用户 --- /usr/bin/wg-quick: line 31: resolvconf: command not found [WireGuard | Debian] - Super User](https://superuser.com/questions/1500691/usr-bin-wg-quick-line-31-resolvconf-command-not-found-wireguard-debian)
[如何在Ubuntu 20.04安装WireGuard VPN | myfreax](https://www.myfreax.com/how-to-set-up-wireguard-vpn-on-ubuntu-20-04/)


