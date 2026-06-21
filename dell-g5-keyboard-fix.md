# Dell G5 5500 键盘背光不亮：完整修复指南

> 问题：AWCC 卸载后键盘灯和底部灯带彻底不亮，重装 AWCC 失败。折腾两天终于解决。

## 设备信息

- **机型**：Dell G5 15 5500
- **灯光芯片**：Alienware AW-ELC（USB HID，VID_187C / PID_0550）
- **区域**：键盘 3 区 RGB + 底部灯带 1 区 RGB
- **键盘接口**：PS/2（`ACPI\DLLK09E2`）

## 根因分析

1. **AWCC 卸载时清空了 AW-ELC 芯片的 SPI 闪存固件初始化**
2. 芯片通信正常（HID 命令全部接受），但 LED 输出级处于断电状态
3. 不重装 AWCC 恢复固件初始化，任何第三方工具（alienfx-tools 等）都无法点亮

## 修复步骤

### 第一步：彻底清理

确认没有 AWCC 残留：
- `C:\Program Files\Alienware\` — 无残留
- `C:\ProgramData\Alienware\` — 无残留
- 注册表 `HKEY_LOCAL_MACHINE\SOFTWARE\Alienware\` — 无残留

### 第二步：下载完整安装包

从 Dell 官网下载，输入你的 Service Tag（笔记本底部贴纸）。

⚠️ **必须下载完整安装包（~1GB），不是小的在线安装器（~6MB）！**

小的在线安装器要从 Dell 服务器现下载组件，网络一抖就失败——这是最常见踩坑点。

### 第三步：安装

- 关闭杀毒软件实时防护
- 右键安装包 → 以管理员身份运行
- 装完自动重启

### 第四步：防止自动更新

管理员 PowerShell：

```powershell
Set-Service -Name DellClientManagementService -StartupType Disabled
Stop-Service -Name DellClientManagementService
```

### 第五步（可选）：背光超时设为常亮

重启，F2 进 BIOS → System Configuration：

| 选项 | 默认 | 推荐 |
| --- | --- | --- |
| Keyboard Backlight Timeout on AC | 10 sec | **Never** |
| Keyboard Backlight Timeout on Battery | 10 sec | **Never** |

## 为什么第三方工具不够？

- **AlienFX-Tools**：500KB，支持 AW-ELC，CLI 操作 —— 但前提是芯片必须先被 AWCC 初始化过
- **Dell-G-Series-Controller**：Linux 项目，靠 USB reset 完成初始化，Windows 上不行
- **自定义脚本（HID）**：协议已逆向，命令全成功 —— 芯片收到但不执行，缺固件级初始化

## 为何第一次重装失败？

大概率是：
1. 用了小的在线安装器（6MB）
2. 或卸载后有注册表残留

## 经验

- Dell G5 的键盘灯光不是纯 HID 控制，芯片需要 AWCC 做一次性固件初始化
- 初始化成功后，第三方工具就都能用了
- BIOS 背光超时默认 10 秒太激进，设 Never 省心

---

*Service Tag：3YN5503 · BIOS：1.14+ · 2026.06*
