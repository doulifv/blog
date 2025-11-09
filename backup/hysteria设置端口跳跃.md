## 安装 iptables-persistent
```
sudo apt update
sudo apt install iptables-persistent
```
默认情况下iptables规则会在重启后失效，因此通过安装iptables-persistent来持久化配置文件
**在安装过程中，系统会提示你是否要保存当前的 iptables 规则，选择“Yes”。**
## 添加 iptables 规则
将ens18网卡31000:32000端口收到的UDP流量传递给30000端口
```
sudo iptables -t nat -A PREROUTING -i ens18 -p udp --dport 31000:32000 -j REDIRECT --to-ports 30000
```
## 持久化 iptables 规则
```
sudo netfilter-persistent save
```
重启netfilter-persistent严重，效果检验是否保持完好
```
systemctl restart netfilter-persistent
```
查看iptables nat规则
```
sudo iptables -t nat -L -v
```
## 还原
删除全部iptables nat规则，使用 **_-F_** 选项可以删除nat表中所有链中的所有规则。
```
iptables -t nat -F
```
---
## 拓展 删除特定NAT规则
查看规则列表及编号
```
iptables -t nat -L --line-numbers
```
删除特定规则，使用 -D 选项，指定要删除的链和规则编号。
删除 POSTROUTING 链中的第 7 条规则
```
iptables -t nat -D POSTROUTING 7 
```
---
参考：[Proxy | Hysteria 2 端口跳跃](https://blog.jianbing.tk/p/hy2-port-hopping/)