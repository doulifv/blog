安装git和make
```
apt install make git -y 
```
使用git拉取代码 、进入文件夹、使用make安装
```
git clone https://github.com/magnific0/wondershaper.git
cd wondershaper
make install
```
通过以下命令可以查看相关运行参数
```
wondershaper -h 
```
>   -a 指定网络适配器
   -d 设置最大下载速率(单位为kbps)
   -u 设置最大上传速率(单位:kbps)
   -p 使用"/etc/systemd/wondershaper.conf"中的预设值
   -f 使用指定的预设文件
   -c 清除适配器的限制
   -s 显示适配器的当前状态

例如:限制网卡ens33的上传和下载为8000kbps每秒(8kbit/s=1kByte)
```
wondershaper -a ens33 -u 8000 -d 8000
```
设置完可以安装speedtest-cli测试一下限速效果
```
apt install speedtest-cli
speedtest #测试网速
```
卸载
```
make uninstall #卸载
reboot #重启
```
