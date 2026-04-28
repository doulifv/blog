https://easytier.cn/guide/installation.html
**一键安装脚本（仅 Linux）**
```
wget -O /tmp/easytier.sh "https://raw.githubusercontent.com/EasyTier/EasyTier/main/script/install.sh" && sudo bash /tmp/easytier.sh install --gh-proxy https://ghfast.top/
```
>Install EasyTier successfully!
>Default Port: 11010(UDP+TCP), Notice allowing in firewall!
>Default Network Name: default, Please change it to your own network name!
>Now EasyTier supports multiple config files. You can create config files in the /opt/easytier/config/ folder
>For more information, please check the documents in official site
>The management example of a single configuration file is as follows
>Status: systemctl status easytier@default
>Start: systemctl start easytier@default
>Restart: systemctl restart easytier@default
>Stop: systemctl stop easytier@default
---
stop easytier@default
```
systemctl stop easytier@default
```
Run in private-mode 
```
easytier-core --private-mode true --network-name {my-network} --network-secret {my-secret}
```
运行后程序会print 配置文件，尝试把它复制到/opt/easytier/config文件夹并且把它运行起来
>instance_name = "default"
dhcp = true
listeners = [
    "tcp://0.0.0.0:11010",
    "udp://0.0.0.0:11010",
    "wg://0.0.0.0:11011",
    "ws://0.0.0.0:11011/",
    "wss://0.0.0.0:11012/",
]
exit_nodes = []
rpc_portal = "0.0.0.0:0"
[[peer]]
uri = "tcp://public.easytier.top:11010"
[network_identity]
network_name = "default"
network_secret = "default"
[flags]
default_protocol = "udp"
dev_name = ""
enable_encryption = true
enable_ipv6 = true
mtu = 1380
latency_first = false
enable_exit_node = false
no_tun = false
use_smoltcp = false
foreign_network_whitelist = "*"
disable_p2p = false
p2p_only = false
relay_all_peer_rpc = false
disable_tcp_hole_punching = false
disable_udp_hole_punching = false

客户端配置[测试运行/临时运行]
```
easytier-core  -i 10.126.126.2   -p udp://服务器IP地址:11010 --network-name {my-network} --network-secret {my-secret} --hostname my-nas
```
> -i 用户手动指定IP地址 也可以为 -d 机器自动通过DHCP获取IP地址 
> -p 是服务器地址包括前面的qiuc://也是由用户输入 
> --network-name 是网络名称 用户输入 （可空）
> --network-secret 是网络密码 （可空）
> --hostname 主机名称 （可空）
> -n 子网代理

客户端安装为服务
```
easytier-cli service install  -i 10.126.126.2   -p udp://服务器IP地址:11010 --network-name {my-network} --network-secret {my-secret} --hostname my-nas
```