

vmwaretools download link
https://packages.vmware.com/tools/releases/
blog link
title: 虚拟机安装VMware Tools时提示缺少KB2919355补丁
https://www.dinghui.org/vmware-tools-microsoft-kb2919355.html

查询VMware kb55798，是由于安装VMware Tools 10.3.x以上版本时，会检测环境是否具备Microsoft Visual C++ 2017 Redistributable先决条件。

解决办法如下（任选其一）：

一、使用 VMware Tools 10.2.x 或更早版本，VMware Tools下载链接汇总如下：
[https://packages.vmware.com/tools/releases/](https://packages.vmware.com/tools/releases/)

二、按照提示安装Windows补丁，注意直接安装 kb2919355 会报错，需要提前安装 kb2919442。

1、下载 Windows8.1-KB2919442-x64.msu

[https://download.microsoft.com/download/C/F/8/CF821C31-38C7-4C5C-89BB-B283059269AF/Windows8.1-KB2919442-x64.msu](https://download.microsoft.com/download/C/F/8/CF821C31-38C7-4C5C-89BB-B283059269AF/Windows8.1-KB2919442-x64.msu)

2、下载 Windows8.1-KB2919355-x64.msu

[https://download.microsoft.com/download/2/5/6/256CCCFB-5341-4A8D-A277-8A81B21A1E35/Windows8.1-KB2919355-x64.msu](https://download.microsoft.com/download/2/5/6/256CCCFB-5341-4A8D-A277-8A81B21A1E35/Windows8.1-KB2919355-x64.msu)

三、更换系统镜像，直接使用含有补丁的较新版本Windows Server 2012 R2。

文中用到的安装包汇总到百度网盘链接:

链接: [https://pan.baidu.com/s/1OUQn91RtsNlUKhx7XdnXMw](https://pan.baidu.com/s/1OUQn91RtsNlUKhx7XdnXMw)
提取码: zr2j
