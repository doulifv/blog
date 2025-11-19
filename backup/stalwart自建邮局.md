## Install on Linux / MacOS   在Linux / MacOS上安装
To install Stalwart on Linux or MacOS, execute the following command in your terminal:
要在 Linux 或 MacOS 上安装 Stalwart，请在终端执行以下命令：
```
curl --proto '=https' --tlsv1.2 -sSf https://get.stalw.art/install.sh -o install.sh
```
Then, execute the installation script as root. The default installation directory is /opt/stalwart. If you want to install Stalwart in a different directory, you can specify the installation directory as an argument:
然后，以root身份执行安装脚本。默认安装目录是 /opt/stalwart。如果你想在另一个目录安装 Stalwart，可以指定安装目录作为参数：
```
sudo sh install.sh /path/to/install
```
If you are planning to use FoundationDB as the backend, add the --fdb parameter to the installation script to download the version compiled with FoundationDB support.
如果你打算用 FoundationDB 作为后端，可以在安装脚本中添加 --fdb 参数，下载支持 FoundationDB 的版本。


## Log in to the web interface 登录网页界面
Once the installation is complete, the installation script will print out the administrator account and password:
安装完成后，安装脚本会打印管理员账户和密码：
```
sudo sh install.sh
✅ Configuration file written to /opt/stalwart/etc/config.toml
🔑 Your administrator account is 'admin' with password 'w95Yuiu36E'.
🎉 Installation complete! Continue the setup at http://yourserver.org:8080/login
```

### 卸载
如果使用脚本默认安装到 /opt/stalwart目录, 可以使用以下步骤彻底卸载: 
停止并禁用 Stalwart 服务
```
systemctl stop stalwart
systemctl disable stalwart
```
删除安装目录
```
rm -rf /opt/stalwart
```
删除 systemd 服务文件
```
rm /etc/systemd/system/stalwart.service
```
重新加载 systemd 配置
```
systemctl daemon-reload
```

---
参考 https://stalw.art/docs/install/platform/linux/