### 安装部署
#### 1、创建本地文件夹映射
```
 mkdir -p /opt/omnibox/data
 cd /opt/omnibox
```
#### 2、Docker Compose配置方式
```
 services:
   omnibox:
     image: lampon/omnibox:latest
     container_name: omnibox
     ports:
       - "7023:7023"
     restart: unless-stopped
     volumes:
       - ./data:/app/data
```
#### 3、Docker命令行方式
```
 docker pull lampon/omnibox:latest
```
##### Docker命令
```
 docker run -d \
    --restart unless-stopped \
    --name omnibox \
    -p 7023:7023 \
    -v $PWD/data:/app/data \
    lampon/omnibox:latest
```
#### 登录管理地址
 **http://NAS_IP:7023/admin**
---
参阅
[OmniBox-一个好用的影视聚合平台,Docker一键部署和使用教程-知乎](https://zhuanlan.zhihu.com/p/2003806837204604477)