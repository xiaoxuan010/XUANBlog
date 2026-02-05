---
title: 在 Windows 上为所有用户安装 Ungoogled-Chromium
# title: Install Ungoogled-Chromium For All Users On Windows
excerpt: 本文介绍如何在 Windows 系统上将 Unggogled-Chromium 安装到 C:\Program Files 目录下，使得所有用户均可使用。
date: 2026-02-05T23:02:08+08:00
tags:
---

默认情况下，Ungoogled-Chromium 安装包会将浏览器安装到当前用户个人目录下（例如 `C:\Users\xiaoxuan010\AppData\Local\Chromium`），本文介绍如何将其安装到系统中（`C:\Program Files\Chromium`），使所有用户均可用。

## 安装步骤

### 下载 Ungoogled-Chromium 安装包

在 GitHub Release 上下载 Ungoogled-Chromium 的 Windows 安装包（例如 `ungoogled-chromium_144.0.7559.109-1.1_installer_x64.exe`）：

{% link https://github.com/ungoogled-software/ungoogled-chromium-windows/releases/latest  %}

### 安装 Ungoogled-Chromium

1. 在安装包的目录下打开终端（例如 {% kbd Shift %} + 右键文件夹空白处，选择“在此处打开 PowerShell 窗口”），执行以下命令：

   ```bash
   .\ungoogled-chromium_144.0.7559.109-1.1_installer_x64.exe --system-level
   ```
   
   将 `ungoogled-chromium_144.0.7559.109-1.1_installer_x64.exe` 替换为你实际下载的安装包的文件名。

2. 在弹出的 UAC 窗口中，点击“是”以允许以管理员权限安装，稍等片刻后即可安装完毕。

   {% image install-command.webp 使用命令行安装，并在 UAC 窗口中确认 fancybox:true %}


3. 查看桌面上自动生成的 Chromium 快捷方式的属性，验证目标路径确实为 Program Files：
    {% image chromium-shortcut-properties.webp Chromium 快捷方式的属性 width:400px padding:16px fancybox:true %}

## 参考资料

{% link https://martinrotter.github.io/it-programming/2016/07/17/install-chromium-system-wide-windows/ desc:true %}
