创建/修改daemon.json文件
```
vi /etc/docker/daemon.json
```
编辑
```
{
  "registry-mirrors": ["https://mirror.ccs.tencentyun.com", "https://hub-mirror.c.163.com"]
}
```
重启验证
```
rc-service docker stop
rc-service docker start
docker info
docker pull hello-world
```