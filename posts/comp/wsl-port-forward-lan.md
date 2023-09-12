---
title: WSL 2 端口转发到局域网
date: 2023-02-05T16:45:00+08:00
category: comp
tags:
  - network
keywords:
  - wsl
---

WSL 2 运行于独立的局域网内，需要一些操作才能将 WSL 2 的端口暴露到 Windows 宿主机所在的局域网。

<!-- more -->

我对计算机网络没有很深入的了解，这一篇主要记录一下是怎么把端口转发跑起来的😟。


## TL;DR

声明一个环境变量 `$env:WSL_IP`，定义两个 PowerShell Cmdlet `Start-WslForward` 和 `Stop-WslForward`。需要安装 [`gsudo`](https://github.com/gerardog/gsudo) 提权。

```powershell
$env:WSL_IP = (wsl -- ip -o -4 -json addr list eth0 `
| ConvertFrom-Json `
| ForEach-Object { $_.addr_info.local } `
| Where-Object { $_ })

function Start-WslForward {
    [CmdletBinding()]
    param (
        [Parameter()]
        [Int16]
        $PortWsl = 3000,
        [Parameter()]
        [Int16]
        $PortLan = 8080,
        [Parameter()]
        [String]
        $DisplayName = $null
    )

    if ([String]::IsNullOrEmpty($DisplayName)) {
        $DisplayName = "WSL-Forward-$($PortLan):$($PortWsl)"
    }

    Invoke-Gsudo {
        netsh interface portproxy add v4tov4 listenport=$using:PortLan connectaddress=$env:WSL_IP connectport=$using:PortWsl
        New-NetFireWallRule -DisplayName $using:DisplayName -Direction Inbound -LocalPort $using:PortLan -Action Allow -Protocol TCP
    }

}

function Stop-WslForward {
    [CmdletBinding()]
    param (
        [Parameter()]
        [Int16]
        $PortWsl = 3000,
        [Parameter()]
        [Int16]
        $PortLan = 8080,
        [Parameter()]
        [string]
        $DisplayName = $null
    )

    if ([String]::IsNullOrEmpty($DisplayName)) {
        $DisplayName = "WSL-Forward-$($PortLan):$($PortWsl)"
    }

    Invoke-Gsudo {
        netsh interface portproxy delete v4tov4 listenport=$using:PortLan
        Remove-NetFirewallRule -DisplayName $using:DisplayName
    }
}
```

用法：

```powershell
# Start
Start-WslForward -PortWsl 3000 -PortLan 8081

# Stop
Stop-WslForward  -PortWsl 3000 -PortLan 8081
```


## WSL 2

首先需要确认 WSL 是否是 WSL 2。WSL 1 的端口是直接暴露在 Windows 宿主上的，而 WSL 2 则位于一个独立的局域网内。

```powershell
wsl --list -v
```

由于 WSL 2 位于独立的局域网，首先要取得它的 IP 地址。以下代码~~粘贴自 SO~~，可以用于有多个 WSL Distro 的情况（比如，除了 WSL 还使用了 Docker for Windows），会打开 WSL 的默认 Distro 并获取其 IP，暴露到 `$env:WSL_IP` 环境变量。

```powershell
$env:WSL_IP = (wsl -- ip -o -4 -json addr list eth0 `
| ConvertFrom-Json `
| ForEach-Object { $_.addr_info.local } `
| Where-Object { $_ })
```

## 使用 `netsh` 转发端口

`netsh` 是 Windows 自带的一个网络命令小工具。可以使用 `interface portproxy` 子命令来监听特定地址（`listenaddress`）的特定端口（`listenport`），将请求转发到一个特定地址（`connectaddress`）的特定端口（`connectport`）。

注意运行需要 Administrator 权限。

```powershell
netsh interface portproxy add v4tov4 listenport=$PortLan connectaddress=$env:WSL_IP connectport=$PortWsl
```

注意 `listenaddress` 要设为缺省值 `*`。如果设为本机地址 `localhost`，会阻止来自局域网的流量，只转发本机流量。同时，`netsh` 会把端口转发的配置写入文件系统，重启机器后仍然生效，因此需要及时关闭：

```powershell
netsh interface portproxy delete v4tov4 listenport=$PortLan
```

## 开放防火墙

```powershell
New-NetFireWallRule -DisplayName 'WSL Port Forward' -Direction Inbound -LocalPort $PortLan -Action Allow -Protocol TCP
```

用以上 PowerShell 脚本可以允许自 `$PortLan` 的入站 TCP 流量进入 Windows。删除防火墙规则的代码不再赘述了。

---

~~这一切都是为了用最好看且最兼容 Windows 的 Linux 桌面发行版——Windows 11~~😵‍💫。
