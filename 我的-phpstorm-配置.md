---
title: 我的 phpstorm 配置
tags:
  - phpstrom
  - 配置
categories:
  - phpstrom
abbrlink: 269f8031
date: 2017-09-22 12:55:39
keywords: [phpstrom,配置]
---
## 前言
感觉 `phpstorm` 很爽，所以抛弃了我心爱的 `atom` 转身投入 `phpstorm` 的怀抱，把我的 `phpstorm` 配置做个笔记，以便在不同环境下使用。

## 下载
在 `Google` 搜索 `phpstorm` 下载。

## 安装
这个直接默认下一步安装就可以。

## 配置
### 视觉美化配置
1. `⌘+,` 来打开配置项。  

2. 选择 `Appearance & Behavior -> Appearance` ，把 `UI Options` 里的 `Theme` 设为 `Darcula` ，打开 `Override default fonts by` ， `Name` 设为 `Fira Code Medium` ，`Size` 设为 15  

3. 选择 `Editor->Font` ，`Font` 设为 `Fira Code` ，`Size` 设为 15 ，`Line spacing` 设为 1.4。  

4. 关闭 `View` 标签下的 `Tool Buttons` 、`Status Bar` 、`Navigation Bar`。  

5. `⌘+⇧+a` 搜索 `Tabs Placement` 设为 `None` 来关闭上方文件标签栏。  

### 功能配置

1. `⌘+⇧+a` 搜索配置项: `plugins` 点击 `Browse repositories` 搜索 `vim` ，下载 `IdeaVim`。

2. `⌘+,` 打开配置项: `keymap` 点击右侧放大镜图标进行快捷键按键搜索，把 `⌘+o` 与 `⌘+⇧+o` 互换。

3. `⌘+,` 打开配置项：`keymap` ，搜索 `⌥+⌘+v` 即 `Tools->vim emulator` 更换为 `⌃+,` 配置 `VIM` 模式切换。

4. `⌘+⇧+a` 搜索配置项: `use soft wraps` 来开启／关闭自动换行。

5. `⌥+F12` 改为 `⌥+t`

6. `⌘+⇧+a` 改为 `⌥+a`

7. `keymap` 搜索 `Database` 添加 `⌥+d`

### 数据库配置
`⌘+⇧+a` 搜索配置项 `database` 打开 `database` 面板，点击 `➕` 号，选择 `Data Source->Mysql` 进行 `Mysql` 配置。注意点击下方 `Driver` 链接下载驱动。
