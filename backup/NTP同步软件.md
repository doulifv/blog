使用Python编写，适合windows7 32位及以上系统，整体功能纯净无广告
软件名称：NTP同步
图形界面（tkinter + 系统托盘）
自定义 NTP 服务器 & 同步间隔
无黑框（使用 pythonw 或打包后自动隐藏）
圆形蓝色托盘图标（带时钟指针）
开机启动开关按钮
通过 Windows API 设置时间
以管理员身份自动提权
兼容 Windows 7 32位 + Python 3.8

编译所需要的依赖
```
pip install ntplib pystray pillow
```
打包成软件
```
pyinstaller --onefile --windowed --name "NTP同步" ntp_gui.py
```
⚠️ 重要提醒

- 首次运行会弹出 UAC 提权窗口（正常现象）

- 若仍提示“设置系统时间失败”，请手动授予当前用户 “更改系统时间”权限（通过 secpol.msc）
- 开机启动功能写入 当前用户 启动项，无需管理员权限

---
ntp_gui.py
```
# ntp_gui.py - NTP同步（Windows 7 32位兼容版）
# 功能：图形界面 + 系统托盘 + 开机启动 + 无黑框 + 权限提取

import os
import sys
import time
import threading
import subprocess
from datetime import datetime, timezone
import tkinter as tk
from tkinter import ttk, messagebox
import ntplib
import pystray
from PIL import Image, ImageDraw

# Windows 注册表（仅 Windows）
try:
    import winreg as reg
except ImportError:
    reg = None

# ----------------------------
# 工具函数
# ----------------------------

def is_admin():
    try:
        return os.getuid() == 0
    except AttributeError:
        import ctypes
        return ctypes.windll.shell32.IsUserAnAdmin() != 0

def run_as_admin():
    """以管理员身份重新运行（优先使用 pythonw 避免黑框）"""
    import ctypes
    executable = sys.executable
    if executable.endswith("python.exe"):
        executable = executable[:-10] + "pythonw.exe"
    script = os.path.abspath(sys.argv[0])
    params = f'"{script}"'
    try:
        ctypes.windll.shell32.ShellExecuteW(None, "runas", executable, params, None, 1)
    except Exception:
        pass
    sys.exit(0)

def set_system_time(dt):
    """使用 Windows API 设置系统时间（支持 Win7 32位）"""
    import ctypes
    from ctypes import wintypes

    class LUID(ctypes.Structure):
        _fields_ = [("LowPart", wintypes.DWORD), ("HighPart", wintypes.LONG)]

    class LUID_AND_ATTRIBUTES(ctypes.Structure):
        _fields_ = [("Luid", LUID), ("Attributes", wintypes.DWORD)]

    class TOKEN_PRIVILEGES(ctypes.Structure):
        _fields_ = [("PrivilegeCount", wintypes.DWORD), ("Privileges", LUID_AND_ATTRIBUTES * 1)]

    def enable_privilege(privilege_name):
        ADVAPI32 = ctypes.windll.advapi32
        KERNEL32 = ctypes.windll.kernel32
        hToken = wintypes.HANDLE()
        if not ADVAPI32.OpenProcessToken(KERNEL32.GetCurrentProcess(), 0x0028, ctypes.byref(hToken)):
            return False
        luid = LUID()
        if not ADVAPI32.LookupPrivilegeValueW(None, privilege_name, ctypes.byref(luid)):
            KERNEL32.CloseHandle(hToken)
            return False
        tp = TOKEN_PRIVILEGES()
        tp.PrivilegeCount = 1
        tp.Privileges[0].Luid = luid
        tp.Privileges[0].Attributes = 2  # SE_PRIVILEGE_ENABLED
        result = ADVAPI32.AdjustTokenPrivileges(hToken, False, ctypes.byref(tp), 0, None, None)
        success = (result != 0) and (KERNEL32.GetLastError() == 0)
        KERNEL32.CloseHandle(hToken)
        return success

    if not enable_privilege("SeSystemtimePrivilege"):
        return False

    class SYSTEMTIME(ctypes.Structure):
        _fields_ = [
            ('wYear', wintypes.WORD),
            ('wMonth', wintypes.WORD),
            ('wDayOfWeek', wintypes.WORD),
            ('wDay', wintypes.WORD),
            ('wHour', wintypes.WORD),
            ('wMinute', wintypes.WORD),
            ('wSecond', wintypes.WORD),
            ('wMilliseconds', wintypes.WORD),
        ]

    st = SYSTEMTIME()
    st.wYear = dt.year
    st.wMonth = dt.month
    st.wDay = dt.day
    st.wHour = dt.hour
    st.wMinute = dt.minute
    st.wSecond = dt.second
    st.wMilliseconds = 0

    return bool(ctypes.windll.kernel32.SetLocalTime(ctypes.byref(st)))

def get_ntp_time(ntp_server):
    client = ntplib.NTPClient()
    try:
        response = client.request(ntp_server, version=3, timeout=5)
        utc_dt = datetime.utcfromtimestamp(response.tx_time).replace(tzinfo=timezone.utc)
        return utc_dt
    except Exception:
        return None

# ----------------------------
# 主应用类
# ----------------------------

class NTPApp:
    def __init__(self, root):
        self.root = root
        self.root.title("NTP同步")
        self.root.geometry("420x260")
        self.root.resizable(False, False)
        self.root.protocol("WM_DELETE_WINDOW", self.hide_to_tray)

        self.ntp_server = "pool.ntp.org"
        self.interval = 3600
        self.running = False
        self.sync_thread = None
        self.tray_icon = None

        self.create_widgets()
        self.create_tray_icon()

        # 初始化开机启动状态
        if reg:
            self.autostart_var.set(self.is_autostart_enabled())

    def create_widgets(self):
        pad = {'padx': 20, 'pady': 5}

        ttk.Label(self.root, text="NTP 服务器地址:").pack(**pad, anchor='w')
        self.server_var = tk.StringVar(value=self.ntp_server)
        ttk.Entry(self.root, textvariable=self.server_var, width=45).pack(**pad)

        ttk.Label(self.root, text="同步间隔（秒，≥10）:").pack(**pad, anchor='w')
        self.interval_var = tk.StringVar(value=str(self.interval))
        ttk.Entry(self.root, textvariable=self.interval_var, width=20).pack(**pad, anchor='w')

        btn_frame = ttk.Frame(self.root)
        btn_frame.pack(pady=8)

        self.sync_btn = ttk.Button(btn_frame, text="立即同步", command=self.manual_sync)
        self.sync_btn.pack(side='left', padx=5)

        self.start_stop_btn = ttk.Button(btn_frame, text="开始自动同步", command=self.toggle_sync)
        self.start_stop_btn.pack(side='left', padx=5)

        self.quit_btn = ttk.Button(btn_frame, text="退出", command=self.quit_app)
        self.quit_btn.pack(side='left', padx=5)

        # 开机启动复选框
        if reg:
            self.autostart_var = tk.BooleanVar()
            self.autostart_check = ttk.Checkbutton(
                self.root,
                text="开机自动启动",
                variable=self.autostart_var,
                command=self.toggle_autostart
            )
            self.autostart_check.pack(**pad, anchor='w')
        else:
            ttk.Label(self.root, text="（开机启动仅支持 Windows）", foreground="gray").pack(**pad, anchor='w')

        self.status_var = tk.StringVar(value="就绪")
        ttk.Label(self.root, textvariable=self.status_var, foreground="gray").pack(pady=5)

    def manual_sync(self):
        self._do_sync()

    def _do_sync(self):
        server = self.server_var.get().strip()
        if not server:
            messagebox.showwarning("输入错误", "请输入 NTP 服务器地址！")
            return

        self.status_var.set("正在同步...")
        self.root.update_idletasks()

        utc_time = get_ntp_time(server)
        if utc_time:
            local_time = utc_time.astimezone().replace(tzinfo=None)
            if set_system_time(local_time):
                self.status_var.set(f"✅ 同步成功: {local_time.strftime('%Y-%m-%d %H:%M:%S')}")
            else:
                self.status_var.set("❌ 设置系统时间失败（请检查权限）")
        else:
            self.status_var.set("❌ 无法连接 NTP 服务器")

    def toggle_sync(self):
        if not self.running:
            try:
                interval = int(self.interval_var.get())
                if interval < 10:
                    messagebox.showwarning("警告", "间隔不能小于10秒")
                    return
                self.interval = interval
                self.ntp_server = self.server_var.get().strip()
                if not self.ntp_server:
                    messagebox.showwarning("警告", "请输入 NTP 服务器地址")
                    return
                self.running = True
                self.start_stop_btn.config(text="停止同步")
                self.sync_thread = threading.Thread(target=self._sync_loop, daemon=True)
                self.sync_thread.start()
                self.status_var.set("自动同步已启动")
            except ValueError:
                messagebox.showerror("错误", "请输入有效的数字作为间隔时间")
        else:
            self.running = False
            self.start_stop_btn.config(text="开始自动同步")
            self.status_var.set("自动同步已停止")

    def _sync_loop(self):
        while self.running:
            self._do_sync()
            for _ in range(self.interval):
                if not self.running:
                    break
                time.sleep(1)

    def hide_to_tray(self):
        self.root.withdraw()

    def show_window(self):
        self.root.deiconify()

    def quit_app(self):
        self.running = False
        if self.tray_icon:
            self.tray_icon.stop()
        self.root.destroy()

    # ----------------------------
    # 系统托盘
    # ----------------------------

    def create_image(self):
        size = 64
        image = Image.new('RGBA', (size, size), (0, 0, 0, 0))
        draw = ImageDraw.Draw(image)
        radius = size // 2 - 4
        center = size // 2
        draw.ellipse((center - radius, center - radius, center + radius, center + radius),
                     fill=(30, 144, 255))  # DodgerBlue
        # 时钟指针
        draw.line([(center, center - radius + 10), (center, center - radius // 2)], fill="white", width=3)
        draw.line([(center + radius // 2, center), (center + radius - 10, center)], fill="white", width=3)
        return image

    def create_tray_icon(self):
        menu = (
            pystray.MenuItem("显示窗口", self.on_tray_show),
            pystray.MenuItem("退出", self.on_tray_quit)
        )
        icon = pystray.Icon("ntp_sync", self.create_image(), "NTP同步", menu)
        self.tray_icon = icon
        threading.Thread(target=icon.run, daemon=True).start()

    def on_tray_show(self, icon, item):
        self.show_window()

    def on_tray_quit(self, icon, item):
        self.quit_app()

    # ----------------------------
    # 开机启动
    # ----------------------------

    def is_autostart_enabled(self):
        if not reg:
            return False
        try:
            key = reg.OpenKey(reg.HKEY_CURRENT_USER,
                              r"Software\Microsoft\Windows\CurrentVersion\Run",
                              0, reg.KEY_READ)
            try:
                value, _ = reg.QueryValueEx(key, "NTP同步")
                reg.CloseKey(key)
                return bool(value)
            except FileNotFoundError:
                reg.CloseKey(key)
                return False
        except Exception:
            return False

    def toggle_autostart(self):
        if not reg:
            return
        try:
            key = reg.OpenKey(reg.HKEY_CURRENT_USER,
                              r"Software\Microsoft\Windows\CurrentVersion\Run",
                              0, reg.KEY_SET_VALUE)

            if self.autostart_var.get():
                if getattr(sys, 'frozen', False):
                    exe_path = os.path.abspath(sys.executable)
                else:
                    pythonw = sys.executable.replace("python.exe", "pythonw.exe")
                    script = os.path.abspath(__file__)
                    exe_path = f'"{pythonw}" "{script}"'
                reg.SetValueEx(key, "NTP同步", 0, reg.REG_SZ, exe_path)
            else:
                try:
                    reg.DeleteValue(key, "NTP同步")
                except FileNotFoundError:
                    pass
            reg.CloseKey(key)
        except Exception as e:
            messagebox.showerror("错误", f"设置开机启动失败:\n{str(e)}")
            self.autostart_var.set(False)

# ----------------------------
# 主程序入口
# ----------------------------

def main():
    if not is_admin():
        run_as_admin()

    root = tk.Tk()
    app = NTPApp(root)
    root.mainloop()

if __name__ == "__main__":
    main()
```