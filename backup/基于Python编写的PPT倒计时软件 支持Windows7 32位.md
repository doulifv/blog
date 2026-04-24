```
import tkinter as tk
from tkinter import colorchooser, simpledialog
import win32com.client
import pythoncom
from comtypes import COMError
import json
import os

# --- 配置文件管理 ---
CONFIG_FILE = "ppt_timer_config.json"

class ConfigManager:
    def __init__(self):
        self.default_config = {
            "x": 200,
            "y": 200,
            "font_size": 48,
            "font_color": "#FF0000",
            "bg_color": "#000000",
            "alpha": 0.85,
            "duration_sec": 300
        }
        self.config = self.load()

    def load(self):
        if os.path.exists(CONFIG_FILE):
            try:
                with open(CONFIG_FILE, "r", encoding="utf-8") as f:
                    data = json.load(f)
                    # 合并配置，防止旧配置文件缺少新字段
                    return {**self.default_config, **data}
            except Exception as e:
                print(f"Config load error: {e}")
                pass
        return self.default_config.copy()

    def save(self):
        try:
            with open(CONFIG_FILE, "w", encoding="utf-8") as f:
                json.dump(self.config, f, indent=4)
        except Exception as e:
            print(f"Config save error: {e}")

# --- 主程序 ---
class PPTAutoTimer:
    def __init__(self, root):
        self.root = root
        self.root.title("PPT Timer")
        self.cfg = ConfigManager()

        # --- 状态变量 ---
        self.time_left = self.cfg.config["duration_sec"]
        self.is_running = False
        self.is_ppt_playing = False
        self.is_overtime = False
        self.overtime_seconds = 0
        
        # --- COM 对象缓存 ---
        self.ppt_app = None

        # --- UI 初始化 ---
        self.setup_window()
        self.create_widgets()
        
        # 初始调整窗口大小
        self.root.after(100, self.adjust_window_size)

        # --- 启动状态监测 (使用 after 代替 threading) ---
        self.check_ppt_status()

    def setup_window(self):
        self.root.overrideredirect(True)
        self.root.attributes('-topmost', True)
        self.root.attributes('-alpha', self.cfg.config["alpha"])
        self.root.configure(bg=self.cfg.config["bg_color"])

        # 尝试设置初始位置
        try:
            self.root.geometry(f"+{self.cfg.config['x']}+{self.cfg.config['y']}")
        except:
            pass

        self.root.bind("<ButtonPress-1>", self.start_move)
        self.root.bind("<B1-Motion>", self.do_move)
        self.root.bind("<Button-3>", self.show_menu)

    def create_widgets(self):
        self.label = tk.Label(
            self.root,
            text="00:00",
            font=("Microsoft YaHei", self.cfg.config["font_size"], "bold"),
            fg=self.cfg.config["font_color"],
            bg=self.cfg.config["bg_color"],
            bd=0,
            padx=0,
            pady=0
        )
        self.label.pack()

        self.menu = tk.Menu(self.root, tearoff=0)
        self.menu.add_command(label="设置时长 (分:秒)", command=self.set_duration)
        self.menu.add_command(label="设置字体大小", command=self.set_font_size)
        self.menu.add_command(label="设置字体颜色", command=self.pick_font_color)
        self.menu.add_command(label="设置背景颜色", command=self.pick_bg_color)
        self.menu.add_command(label="设置透明度", command=self.set_opacity)
        
        self.menu.add_separator()
        self.menu.add_command(label="退出", command=self.exit_app)

    def exit_app(self):
        # 保存当前窗口位置
        try:
            self.cfg.config['x'] = self.root.winfo_x()
            self.cfg.config['y'] = self.root.winfo_y()
            self.cfg.save()
        except:
            pass
        self.root.destroy()

    def set_opacity(self):
        current_percent = int(self.cfg.config["alpha"] * 100)
        val = simpledialog.askinteger("设置透明度", "请输入不透明度百分比 (10-100):", initialvalue=current_percent, minvalue=10, maxvalue=100, parent=self.root)
        if val is not None:
            alpha_value = val / 100.0
            self.cfg.config["alpha"] = alpha_value
            self.cfg.save()
            self.root.attributes('-alpha', alpha_value)

    # --- 核心逻辑：监测 PPT (优化版：单线程轮询) ---
    def check_ppt_status(self):
        is_playing_now = False
        
        try:
            # 1. 尝试获取或复用 PPT 对象
            if self.ppt_app is None:
                self.ppt_app = win32com.client.GetActiveObject("PowerPoint.Application")
            
            # 2. 检查放映窗口数量
            # 注意：这里需要确保 Count 属性存在，防止极罕见的竞态条件
            if self.ppt_app.SlideShowWindows and self.ppt_app.SlideShowWindows.Count > 0:
                is_playing_now = True
                
        except (COMError, pythoncom.com_error, AttributeError):
            # 如果 PPT 没开，或者 COM 连接断开，重置对象引用
            self.ppt_app = None
            is_playing_now = False
        except Exception:
            # 捕获其他未知错误，防止程序崩溃
            is_playing_now = False

        # --- 状态机逻辑 ---
        if is_playing_now and not self.is_ppt_playing:
            # 状态：从 "没播放" 变为 "正在播放" -> 启动计时
            self.is_ppt_playing = True
            self.start_timer()
        elif not is_playing_now and self.is_ppt_playing:
            # 状态：从 "正在播放" 变为 "没播放" -> 停止/重置
            self.is_ppt_playing = False
            self.reset_timer()

        # 无论状态如何，1秒后再次检查 (1000ms)
        self.root.after(1000, self.check_ppt_status)

    def start_timer(self):
        self.is_running = True
        self.is_overtime = False
        self.overtime_seconds = 0
        self.time_left = self.cfg.config["duration_sec"]
        self.update_display()
        self.run_timer_step()

    def run_timer_step(self):
        if self.is_running:
            # 只有当 PPT 还在播放时才继续计时
            if self.is_ppt_playing:
                if self.time_left > 0:
                    # --- 倒计时阶段 ---
                    self.time_left -= 1
                    self.is_overtime = False
                else:
                    # --- 超时阶段 ---
                    self.is_overtime = True
                    self.overtime_seconds += 1

                self.update_display()
                # 1秒后继续下一步
                self.root.after(1000, self.run_timer_step)
            else:
                # 如果计时器还在运行但 PPT 停止了（通过菜单强制停止等情况）
                pass

    def reset_timer(self):
        self.is_running = False
        self.overtime_seconds = 0
        # 这里的 update_display 是为了让界面立即变回 00:00 或初始状态
        # 如果希望 PPT 退出后保留最后的时间，可以去掉下面这行
        self.update_display()

    def update_display(self):
        if self.is_overtime:
            # --- 超时显示逻辑 ---
            mins, secs = divmod(self.overtime_seconds, 60)
            time_str = f"+{mins:02d}:{secs:02d}"
        else:
            # --- 倒计时显示逻辑 ---
            mins, secs = divmod(self.time_left, 60)
            time_str = f"{mins:02d}:{secs:02d}"

        self.label.config(text=time_str, fg=self.cfg.config["font_color"])
        self.adjust_window_size()

    def adjust_window_size(self):
        try:
            self.label.update_idletasks()
            text_width = self.label.winfo_reqwidth()
            text_height = self.label.winfo_reqheight()

            padding_x = 20
            padding_y = 10

            final_width = text_width + padding_x
            final_height = text_height + padding_y

            # 获取当前坐标以保持位置不变
            x = self.root.winfo_x()
            y = self.root.winfo_y()

            final_width = max(final_width, 50)
            final_height = max(final_height, 30)

            self.root.geometry(f"{final_width}x{final_height}+{x}+{y}")
        except:
            pass

    # --- 交互功能 ---
    def set_duration(self):
        input_str = simpledialog.askstring("设置时长", "请输入倒计时时长 (支持格式 MM:SS 或纯秒数):", initialvalue=str(self.cfg.config["duration_sec"]), parent=self.root)
        if input_str:
            try:
                input_str = input_str.replace("：", ":")
                if ":" in input_str:
                    m, s = map(int, input_str.split(":"))
                    total_seconds = m * 60 + s
                else:
                    total_seconds = int(input_str)

                if total_seconds > 0:
                    self.cfg.config["duration_sec"] = total_seconds
                    self.cfg.save()
                    # 如果当前没有在运行，立即更新显示
                    if not self.is_running:
                        self.time_left = total_seconds
                        self.is_overtime = False
                        self.overtime_seconds = 0
                        self.update_display()
            except ValueError:
                pass

    def set_font_size(self):
        val = simpledialog.askinteger("字体大小", "输入字体大小 (像素):", initialvalue=self.cfg.config["font_size"], minvalue=10, maxvalue=200, parent=self.root)
        if val:
            self.cfg.config["font_size"] = val
            self.label.config(font=("Microsoft YaHei", val, "bold"))
            self.cfg.save()
            self.adjust_window_size()

    def pick_font_color(self):
        color = colorchooser.askcolor(color=self.cfg.config["font_color"], parent=self.root)[1]
        if color:
            self.cfg.config["font_color"] = color
            self.label.config(fg=color)
            self.cfg.save()

    def pick_bg_color(self):
        color = colorchooser.askcolor(color=self.cfg.config["bg_color"], parent=self.root)[1]
        if color:
            self.cfg.config["bg_color"] = color
            self.root.configure(bg=color)
            self.label.config(bg=color)
            self.cfg.save()

    def show_menu(self, event):
        self.menu.tk_popup(event.x_root, event.y_root)

    def start_move(self, event):
        self.x = event.x
        self.y = event.y

    def do_move(self, event):
        deltax = event.x - self.x
        deltay = event.y - self.y
        x = self.root.winfo_x() + deltax
        y = self.root.winfo_y() + deltay
        self.root.geometry(f"+{x}+{y}")

if __name__ == "__main__":
    # 尝试处理 DPI 设置 (兼容 Win7/10/11)
    try:
        from ctypes import windll
        # SetProcessDpiAwareness(1) -> SYSTEM_DPI_AWARE
        windll.shcore.SetProcessDpiAwareness(1)
    except Exception:
        # Win7 可能没有 shcore 或者权限不足，忽略错误
        pass

    root = tk.Tk()
    app = PPTAutoTimer(root)
    root.mainloop()
```