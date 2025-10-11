安装依赖
```
apt install open-vm-tools open-vm-tools-desktop
```
查看共享文件夹[设置的共享名]
```
vmware-hgfsclient
```
设置共享文件夹
```
vmhgfs-fuse .host:/文件夹名/ /mnt/hgfs -o allow_other
```
```
cd /mnt/hgfs
```