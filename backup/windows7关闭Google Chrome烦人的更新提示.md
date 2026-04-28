使用TXT文本编辑器
新建注册表文件，更给TXT后缀为 **.reg** 
closeChromeUpdateWarning.reg
```
Windows Registry Editor Version 5.00

[HKEY_CURRENT_USER\Software\Policies\Google\Chrome]
"SuppressUnsupportedOSWarning"=dword:00000001
```
双击添加到注册表即可

---
[如何关闭“若要接收后续 google chrome 更新,您需使用 windows 10 或更高版本”](https://www.jianshu.com/p/d1001a98fb77)
  