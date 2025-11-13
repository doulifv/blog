查看IP信息
```
curl ip.im/info
```
IP质检
```
bash <(curl -sL IP.Check.Place)
```
```
bash <(curl -sL Net.Check.Place)
```
233boy sing-box一键脚本
```
bash <(wget -qO- -o- https://github.com/233boy/sing-box/raw/main/install.sh)
```
设置变量ip=192.168.1.2适用于申请ssl证书与本地ip不一致的情况
```
export ip=192.168.1.2
```
http代理
```
export http_proxy=http://192.168.1.2:12334
```
socks5代理
```
export http_proxy="socks5://127.0.0.1:1080"
```
**more writing...is soon**