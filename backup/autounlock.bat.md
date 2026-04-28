```
@echo off
setlocal enabledelayedexpansion

:: 设置要监控的目录（当前目录）
set "watch_dir=H:\auto\file"

:: 设置要执行的程序
set "target_exe=H:\auto\et.exe"

:: 初始化文件列表
dir /b /a-d "%watch_dir%" > oldfiles.tmp 2>nul

:loop
:: 使用ping命令实现2秒等待（ping 本机5次，每次间隔1秒，共约5秒）
ping -n 5 127.0.0.1 > nul

:: 获取当前文件列表
dir /b /a-d "%watch_dir%" > newfiles.tmp 2>nul

:: 比较文件差异
fc /b oldfiles.tmp newfiles.tmp >nul
if %errorlevel% equ 1 (
    echo 检测到新文件，正在执行 %target_exe% ...
    
    :: 启动程序并发送回车键
    start "" /wait cmd /c echo. | "%target_exe%"
    
    echo 程序执行完成，继续监控...
)

:: 更新文件列表
move /y newfiles.tmp oldfiles.tmp >nul
goto loop
```