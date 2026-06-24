
https://github.com/doulifv/ntp
---

✅基于Python开发 完全开源无广告
✅ 开机自启 自动开始定时同步 
✅ 逻辑简单 运行流畅不卡顿
✅ 自动提权 自动获取管理员权限
✅ 高度自定义 自定义NTP服务器地址 自定义同步间隔
✅ Win7 32 位完美兼容

---
依赖
```
pip install pyqt5 ntplib pywin32
```
打包命令
```
pyinstaller -w -F --uac-admin --onefile --windowed --icon="NONE" main.py
```