#####以Windows7为例
#####显示当前正在使用的端口转发规则
```
netsh interface portproxy show all
```
#####添加端口转发
```
netsh interface portproxy add v4tov4 listenaddress=192.168.1.214 listenport=45622 connectaddress=192.168.6.128 connectport=45622

#源主机192.168.1.214:45622 转发到 192.168.6.128:45622
```
#####删除转发规则
```
netsh interface portproxy delete v4tov4 listenaddress=192.168.1.214 listenport=45622 
```
---
![image.png](https://upload-images.jianshu.io/upload_images/27021437-58e517f1e8f128c1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

[netsh windows7 自带端口映射（转发）使用教程-CSDN](https://blog.csdn.net/u010953692/article/details/77045211)
[windows命令行下用netsh实现端口转发(端口映射)-CSDN](https://blog.csdn.net/dogquill/article/details/50143925)