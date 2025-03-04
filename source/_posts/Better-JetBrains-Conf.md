---
title: 优化 JetBrains IDE 配置，适合从 VSCode 迁移的用户
excerpt: 整理一些从 VSCode 迁移到 DevEco Studio 的配置优化，让你的打代码体验更丝滑。这些配置一般也适用于其他 JetBrains IDE，如 IntelliJ IDEA 等。
date: 2025-03-04 11:38:53
tags:
  - Coding
  - IDE
  - VSCode
  - JetBrains
  - DevEco
---

## 问题引入

学习鸿蒙原生开发，不得已从 Visual Studio Code 换用到了 DevEco Studio，很多在 VSCode 中习以为常的操作都不适用了，而网上暂时没有找到系统的优化教程，因此本文将整理一些适用于从 VSCode 迁移到 DevEco Studio 的配置优化，这些配置有些来源于网络，有些是我自己摸索出来的，希望能帮助到有需要的人。

由于 DevEco 是基于 IntelliJ IDEA 开源版本改的，因此本文的配置应该也适用于大部分 JetBrains IDE，如 IntelliJ IDEA 等。

## 优化配置

### 1. 改中文

- 默认表现：在当前版本（DevEco Studio 5.0.2 Release）下，DevEco 默认是英文界面。
- 预期表现：将 DevEco 界面改为中文。

实际上，当前版本的 DevEco 已预装了中文语言包，只是默认没有启用，我们只需要手动启用即可。

打开 `Settings > Plugins > Installed` 页面，找到 `Chinese(Simplified)`，点击 `Enable` 启用，然后按照提示重启 IDE 即可。

<!-- ![在 Settings 页面启用中文插件的截图](enable-chinese-plugin.png) -->

{% image enable-chinese-plugin.png 在 Setting 页面启用中文插件 fancybox:true %}

### 2. 快捷键

- 默认表现：DevEco 的快捷键与 JetBrains 系列 IDE 大致相同。
- 预期表现：将 DevEco 的快捷键改为 VSCode 的快捷键。

1. 打开“设置 > 插件”页面，安装并启用 `VSCode Keymap` 插件。
   {% image enable-vscode-keymap-plugin.png 安装并启用 VSCode Keymap 插件 fancybox:true %}

2. 转到“设置 > 快捷键”页面，选择 `VSCode` 作为 Keymap。
   {% image set-vscode-keymap.png 选择 VSCode 作为 Keymap width:380px fancybox:true %}

3. 修改以下快捷键选项，使其更接近 VSCode 的编码体验：
   - 编辑器操作
     - Enter：默认快捷键为 {% kbd Enter %}，添加一个快捷键为 {% kbd Shift %} + {% kbd Enter %}。这个快捷键的用途是，有时候打字较快，例如输入 `{` 符号后快速换行，{% kbd Shift %} 键可能还未完全松开，这时候会导致换行没反应。

### 3. 字体

- 默认表现：DevEco 默认采用 JetBrains Mono 字体。
- 预期表现：将 DevEco 的字体改为 Consolas。

打开“设置 > 编辑器 > 字体”页面，将字体改为 Consolas，大小改为 16.0。此项可以按照个人喜好调整。

### 4. 编辑器滚动

- 默认表现：DevEco 默认的滚动我不太喜欢，感觉控制起来不够精确，尤其是混合使用鼠标滚轮和触摸板滚动时。
- 预期表现：将 DevEco 的滚动风格改为 VSCode 的滚动风格。

{% note 不推荐的做法 有部分文章建议在“设置 > 外观和行为 > 外观 > UI选项”中关闭“平滑滚动”选项。经过测试，该选项并没有解决关键问题，反而导致触摸板滚动操作起来更加别扭了。请根据实际情况调整该选项，我并不推荐关闭。 color:amber %}

双击 {% kbd Shift %}，或按下 {% kbd Ctrl %} + {% kbd Shift %} + {% kbd P %}（如果已启用 `VSCode Keymap`），在搜索框中输入“平滑”，打开“平滑滚动选项”，关闭“动画平滑滚动”选项，确认。

{% image set-scrolling.png 关闭“动画平滑滚动”选项 width:300px %}

### 5. 注释符号位置

- 默认表现：在之前使用 IntelliJ IDEA 编写 Java 代码，使用快捷键切换行注释时，默认的注释符号 `//` 的位置是在行首。
- 预期表现：改为 VSCode 的风格，即将注释也对其当前块的缩进。

打开“设置 > 编辑器 > 代码样式”页面，选择需要更改风格的语言，在“代码生成”选项卡中取消勾选“第一列的行注释”“第一列的列注释”等选项，启用“在注释开始处添加空格”。

{% image set-comment-style.png 调整注释符号样式 %}

### 6. 注释行为

- 默认表现：使用快捷键切换行注释时，无论是注释还是取消注释，都会自动跳转到下一行。
- 预期表现：改为 VSCode 的风格，即注释后保持在当前行。

打开“设置 > 高级设置”页面，搜索并关闭“使用行注释操作在注释后向下移动光标”选项。

{% image disable-cursor-moving-option.png 关闭“使用行注释操作在注释后向下移动光标”选项 width:380px %}
