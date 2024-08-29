---
layout: post
title: 适用于 Windows 的软件包管理器 - Scoop
date: 2023-04-22 05:14 +0000
categories:
- 系统
- Windows
tags:
- scoop
---
[Scoop](https://scoop.sh)，无意中发现的一款 Windows 下的软件包管理器，通过命令在 Windows 开发环境直接快速安装升级常用软件工具，避免到处找安装包下载并手动安装。


## 安装方法
```powershell
# 可选，首次运行远程脚本时执行并允许
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser

# 普通权限账号安装
irm get.scoop.sh | iex

# 管理员权限账号安装，并指定安装目录
iex "& {$(irm get.scoop.sh)} -RunAsAdmin -ScoopDir 'D:\scoop'"
```

添加常用的仓库：
```powershell
# 先安装 git
scoop install git

scoop bucket add extras
scoop bucket add java
```

默认仓库下载可能较慢，有条件可以设置代理：
```powershell
# Socks5
$env:all_proxy="socks5://127.0.0.1:1080"

# HTTP
$env:all_proxy="http://127.0.0.1:8080"

# 仅代理 HTTP/HTTPS 访问请求
$env:HTTPS_PROXY="http://127.0.0.1:8080"
$env:HTTP_PROXY="http://127.0.0.1:8080"
```

```bat
set https_proxy="http://127.0.0.1:8080"
set http_proxy="http://127.0.0.1:8080"
```

## 使用方法
安装软件包：
```powershell
scoop install googlechrome
scoop install openjdk17 maven nodejs yarn
```

查看已安装软件包：
```powershell
scoop list
```

批量升级已安装软件包：
```powershell
scoop update -a
```

清除历史版本和缓存：
```powershell
scoop cleanup *
scoop cache rm *
```

更多使用方法：
```powershell
scoop help
```
