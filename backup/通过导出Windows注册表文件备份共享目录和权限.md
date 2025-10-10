export(admin).bat
```
@echo off
set rf=.\shareConfig.reg

reg export HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\lanmanserver\Shares %rf%
pause
```
import(admin).bat
```
@echo off
set rf=.\shareConfig.reg

reg import %rf%
net stop server
net start server
net share
pause
```
