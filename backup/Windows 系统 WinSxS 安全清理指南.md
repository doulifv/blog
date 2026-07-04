### 严禁手动删除 WinSxS 内部任何文件 / 文件夹
### 如果你是Windows 10系统
#### 常规深度清理（可卸载更新）
```
Dism /Online /Cleanup-Image /StartComponentCleanup
```
- 作用：自动清理超过 30 天的旧组件、过期补丁备份；
- 耗时 5–20 分钟，窗口卡住不动是正常扫描，不要关闭；
- 清理后依旧可以卸载 Windows 更新，风险极低。
#### 极限清理 /ResetBase 系统稳定 1 个月以上再用，不可逆
```
Dism /Online /Cleanup-Image /StartComponentCleanup /ResetBase
```
***执行后所有已安装补丁无法卸载回滚，遇到更新 bug 只能重装系统 / 恢复备份***
#### 查看可清理空间（分析组件存储）
```
Dism /Online /Cleanup-Image /AnalyzeComponentStore
``` 
- 会输出 WinSxS 真实占用、可回收备份大小，判断是否需要清理
#### 修复系统（防止组件损坏）
```
sfc /scannow
```
- 检查修复系统故障

---

### 如果你是windows7系统
#### 清理 SP1 备份
```
Dism /Online /Cleanup-Image /SpSuperseded
```
- 用途：删除 SP1 安装时留下的旧系统组件备份，经常能释放 5~10GB；
- 执行完必须重启电脑，重启时会配置更新，不要断电。

#### 常规组件清理（可卸载更新）
```
Dism /Online /Cleanup-Image /StartComponentCleanup
```
- 自动清理过期组件、冗余补丁文件，保留更新回滚能力
#### 极限清理 /ResetBase（空间极度不足再用，不可逆使用后不可卸载更新）
```
Dism /Online /Cleanup-Image /StartComponentCleanup /ResetBase
```
- 警告：执行后所有已安装补丁无法卸载，遇到更新故障只能重装系统 / 恢复备份；仅适合系统稳定 1 个月以上、C 盘严重告急。
#### 查看可清理空间（分析组件存储同windows10）
```
Dism /Online /Cleanup-Image /AnalyzeComponentStore
```
#### 修复系统（防止组件损坏）
```
sfc /scannow
```
- 检查修复系统故障