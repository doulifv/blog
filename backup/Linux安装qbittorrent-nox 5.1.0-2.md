## 更新/安装/查看日志
```
sudo apt update
sudo apt install qbittorrent-nox
journalctl -fu qbittorrent-nox
```
>******** Information ********
>Nov 29 23:25:30 localhost qbittorrent-nox[145969]: To control qBittorrent, access the WebUI at: http://>localhost:8000
>Nov 29 23:25:30 localhost qbittorrent-nox[145969]: The WebUI administrator username is: **admin**
>Nov 29 23:25:30 localhost qbittorrent-nox[145969]: The WebUI administrator password was not set. A >temporary password is provided for this session: **HH67hgg6**
>Nov 29 23:25:30 localhost qbittorrent-nox[145969]: You should set your own password in program >preferences.
## 不知道密码?
如果没有见到密码输出，可以手动修改配置文件
```
vi /etc/systemd/system/qbittorrent-nox.service
```
添加WorkingDirectory
```
[Unit]
Description=qBittorrent Command Line Client
After=network.target

[Service]
Type=simple
User=root
Group=root
UMask=007
WorkingDirectory=/root/.config/qBittorrent
ExecStart=/usr/bin/qbittorrent-nox --webui-port=8080
Restart=on-failure

[Install]
WantedBy=multi-user.target
```
编辑配置文件
```
vi ~/.config/qBittorrent/qBittorrent.conf
```
添加以下内容
```
[Preferences]
WebUI\User=admin
WebUI\Password_PBKDF2="@ByteArray(ARQ77eY1NUZaQsuDHbIMCA==:0WMRkYTUWVT9wVvdDtHAjU9b3b7uB8NR1Gur2hmQCvCDpm39Q+PsJRJPaCU51dEiz+dTzh8qbPsL8WkFljQYFQ==)"
```
重载/重启服务
```
systemctl stop qbittorrent-nox
systemctl daemon-reload
systemctl start qbittorrent-nox
```
账户admin 
密码adminadmin

参考
[apt安装最新版qbittorrent-nox 5.0.1，并修改密码](https://www.thsink.com/notes/1842/)