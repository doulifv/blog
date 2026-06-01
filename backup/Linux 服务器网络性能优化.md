```
bash <(curl -sL https://raw.githubusercontent.com/666shen/tcp-dashboard/main/tcp.sh)
```
功能模块 | 对应菜单选项 | 作用与效果
-- | -- | --
协议栈优化 | 选项 1 (IPv4优先) | 修改 gai.conf，强制系统优先使用 IPv4，避免 IPv6 绕路导致的连接卡顿。
拥塞算法 | 选项 2 (BBR+FQ) | 启用 Google 的 BBR 拥塞控制算法和 FQ 排队规则，加速单线程下载速度，降低丢包。
内核深度调优 | 选项 3 (生产级调优) | 最核心部分。极大扩充连接队列、文件句柄数，并针对 UDP (QUIC/Reality) 和 TCP 进行低延迟优化。
网卡多队列 | 选项 4 (NIC均衡) | 利用 ethtool 和 RPS（接收包 steering），将网卡中断分发到多个 CPU 核心，解决单核瓶颈。
---
## 如何“手搓”实现类似效果（操作前请确保你有 root 权限）
### 1. 手动实现“生产级内核调优” (对应选项 3)
提升服务器并发能力和抗丢包能力。

- 创建配置文件：

```
vi /etc/sysctl.d/99-network-performance.conf
```

- 粘贴以下内容（根据你的内存大小，适当调整 rmem_max 的值）：

```
# --- 基础队列算法 ---
net.core.default_qdisc = fq
net.ipv4.tcp_congestion_control = bbr

# --- 缓冲区与容量优化 ---
net.core.somaxconn = 65535
net.core.netdev_max_backlog = 65535
net.ipv4.tcp_max_syn_backlog = 16384
net.ipv4.ip_local_port_range = 1024 65535

# 注意：以下数值是基于内存总量的 5%，如果你内存是 2G，请改为约 107374182 (100MB)
net.core.rmem_max = 214748364  # 约 200MB (针对 4G 内存 VPS)
net.core.wmem_max = 214748364
net.ipv4.tcp_rmem = 4096 87380 214748364
net.ipv4.tcp_wmem = 65536 65536 214748364

# --- 低延迟与抗丢包调优 ---
net.ipv4.tcp_notsent_lowat = 16384     # 降低网页首包延迟 (TTFB)
net.ipv4.tcp_mtu_probing = 1           # 开启 MTU 探测，防止黑洞
net.ipv4.udp_rmem_min = 16384          # 扩容 UDP 缓冲区 (针对 Hysteria2/QUIC)
net.ipv4.udp_wmem_min = 16384
net.ipv4.tcp_ecn = 1                   # 开启 ECN，拥塞时不丢包只打标记
net.ipv4.tcp_max_orphans = 32768       # 限制孤儿连接，防止内存耗尽

# --- 连接稳定性 ---
net.ipv4.tcp_slow_start_after_idle = 0
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_fin_timeout = 15
net.ipv4.tcp_retries2 = 8
net.ipv4.tcp_fastopen = 3
```
- 使配置生效
```
sysctl --system
```
### 2. 手动实现“文件句柄限制” (对应选项 3 的副作用)
脚本会修改系统的文件打开限制，防止高并发时“too many open files”错误。
- 创建 limits 配置：
```
vi /etc/security/limits.d/99-network-performance.conf
```
- 写入：
```
* soft nofile 1048576
* hard nofile 1048576
* soft nproc 65535
* hard nproc 65535
```
注意： 修改此配置后，需要重新登录 SSH 或重启系统才能完全生效。
### 3. 手动实现“网卡多队列” (对应选项 4)
如果你的 VPS 是多核 CPU，这一步能极大降低 CPU 占用率，防止网卡中断只压在一个核上。
- 安装工具：
```
# Debian/Ubuntu
apt-get install -y ethtool
# CentOS/RHEL
yum install -y ethtool
```
- 手动执行优化命令（假设你的网卡名是 eth0，核心数是 4）：
```
# 获取网卡支持的最大队列数 (通常在 512 到 4096 之间)
MAX_RX=$(ethtool -g eth0 | grep -A 5 "Pre-set maximums" | grep "RX:" | awk '{print $2}')

# 设置接收(RX)和发送(TX)队列数为最大值
ethtool -G eth0 rx ${MAX_RX:-1024} tx ${MAX_RX:-1024}

# 开启 RPS (接收包 steering)，将中断分发到所有 CPU 核心
# 0xf 代表 4 个核心 (1111 in binary), 0xff 代表 8 个核心
for rps_file in /sys/class/net/eth0/queues/rx-*/rps_cpus; do
    echo "f" > "$rps_file"
done

# 开启 RFS (接收流 steering)
for rfc_file in /sys/class/net/eth0/queues/rx-*/rps_flow_cnt; do
    echo "4096" > "$rfc_file"
done

# 设置全局套接字流条目
sysctl -w net.core.rps_sock_flow_entries=32768
```
### 4. 手动实现“IPv4 优先” (对应选项 1)
- 编辑 gai.conf：
```
vi /etc/gai.conf
```
- 在文件末尾添加一行：
```
precedence ::ffff:0:0/96 100
```
(这行命令的意思是：当 IPv4 和 IPv6 地址都存在时，优先使用 IPv4)
---
## 如何“回退”：
如果你配置后出现问题，脚本的选项 5（rollback_tcp_tune）其实就是执行了以下操作：
- 删除 /etc/sysctl.d/99-network-performance.conf。
- 删除 /etc/security/limits.d/99-network-performance.conf。
- 运行 sysctl -w net.ipv4.tcp_congestion_control=cubic 恢复默认算法。
