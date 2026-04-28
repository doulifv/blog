```
vi /etc/apk/repositories
```
>#http://mirrors.aliyun.com/alpine/v3.21/community
>取消community源的注释
```
apk update
apk add docker
```
添加开机自启动、启动docker服务
```
rc-update add docker boot
service docker start
```
> alpine:~# rc-update add docker boot
>   * service docker added to runlevel boot
>alpine:~# service docker start
>   * Caching service dependencies ...                                                         [ ok ]
>   * /var/log/docker.log: creating file
>   * /var/log/docker.log: correcting owner
>   * Starting Docker Daemon ...                                                               [ ok  ]
> alpine:~# docker -v
>Docker version 27.3.1