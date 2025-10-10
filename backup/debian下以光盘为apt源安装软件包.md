对于一些停止维护的debian版本网络软件包资源大多停止维护，此时可以通过挂载ISO安装光盘实现本地软件包安装
```
apt-cdrom add
```
系统会提示将你要添加为源的光盘放入光驱
然后enter继续，就自动添加好了
>root@214:~# **apt-cdrom add**
Using CD-ROM mount point /media/cdrom/
Unmounting CD-ROM...
Waiting for disc...
**Please insert a Disc in the drive and press [Enter]**
Mounting CD-ROM...
Identifying... [3499991a1c974d34b1f07cea4df324ac-2]
Scanning disc for index files...
Found 2 package indexes, 0 source indexes, 2 translation indexes and 0 signatures
This disc is called: 
'Debian GNU/Linux 9.13.0 _Stretch_ - Official amd64 DVD Binary-1 20200718-11:07'
Reading Package Indexes... Done
Reading Translation Indexes... Done
Writing new source list
Source list entries for this disc are:
***deb cdrom:[Debian GNU/Linux 9.13.0 _Stretch_ - Official amd64 DVD Binary-1 20200718-11:07]/ stretch contrib main***
Unmounting CD-ROM...
Repeat this process for the rest of the CDs in your set.
---
[捷杰耶夫csdn-](https://blog.csdn.net/whw8007 "捷杰耶夫")-[debian下以光盘为apt源安装软件包](https://blog.csdn.net/whw8007/article/details/8833083)