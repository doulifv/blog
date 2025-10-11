
>root@debian:~# apt update
Get:1 http://repo.mysql.com/apt/debian buster InRelease [22.1 kB]     
Hit:2 http://mirrors4.tuna.tsinghua.edu.cn/debian buster InRelease    
Hit:3 http://mirrors4.tuna.tsinghua.edu.cn/debian buster-updates InRelease
Get:4 http://mirrors4.tuna.tsinghua.edu.cn/debian buster-backports InRelease [51.4 kB]
Err:1 http://repo.mysql.com/apt/debian buster InRelease
  The following signatures couldn't be verified because the public key is not available: NO_PUBKEY **467B942D3A79BD29**
Get:5 http://mirrors4.tuna.tsinghua.edu.cn/debian-security buster/updates InRelease [34.8 kB]
Reading package lists... Done      
W: GPG error: http://repo.mysql.com/apt/debian buster InRelease: The following signatures couldn't be verified because the public key is not available: NO_PUBKEY 467B942D3A79BD29
E: The repository 'http://repo.mysql.com/apt/debian buster InRelease' is not signed.
N: Updating from such a repository can't be done securely, and is therefore disabled by default.
N: See apt-secure(8) manpage for repository creation and user configuration details.

添加
```
apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 467B942D3A79BD29
```
>root@debian:~# apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 467B942D3A79BD29
Executing: /tmp/apt-key-gpghome.BKquJCirKG/gpg.1.sh --keyserver hkp://keyserver.ubuntu.com:80 --recv 467B942D3A79BD29
gpg: key 467B942D3A79BD29: public key "MySQL Release Engineering <mysql-build@oss.oracle.com>" imported
gpg: Total number processed: 1
gpg:               imported: 1

此时再更新就没有问题了

参考:[#解决 由于没有公钥，无法验证下列签名 :NO_PUBKEY-博客园](https://www.cnblogs.com/2205254761qq/p/11863928.html)
