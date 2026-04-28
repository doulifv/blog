临时路由
```
route add <目标网络> mask <子网掩码> <网关地址>
```
永久路由-[跃点可缺省]
```
route -p add <目标网络> mask <子网掩码> <网关地址> metric <跃点数>
```
路由表查看
```
route print 
```
测试路由
```
tracert <IP地址> 
```
静态路由删除
```
route delete 
```
改变路由
```
route change <目标网络> mask <子网掩码> <新网关> metric <新跃点数> 
```