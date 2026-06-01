<div align="center">

# upup-vpn-skill 🔒

**私人 VPN 一键管理，Claude Code 自动化技能**

Manage your private VPN infrastructure with Claude Code automation.

[![GitHub Stars](https://img.shields.io/github/stars/iTrimut/upup-vpn-skill?style=social)](https://github.com/iTrimut/upup-vpn-skill/stargazers)
[![GitHub Issues](https://img.shields.io/github/issues/iTrimut/upup-vpn-skill)](https://github.com/iTrimut/upup-vpn-skill/issues)
[![GitHub Forks](https://img.shields.io/github/forks/iTrimut/upup-vpn-skill?style=social)](https://github.com/iTrimut/upup-vpn-skill/network/members)
[![License](https://img.shields.io/github/license/iTrimut/upup-vpn-skill)](LICENSE)

[English](README_EN.md) | [中文](README.md)

</div>

---

## 项目简介

upup-vpn-skill 是一个 **Claude Code Skill**，用于管理个人 VPN 基础设施。当用户在 Claude Code 中提到 VPN 相关操作时，自动加载此技能并执行对应任务。

支持 macOS、iPhone、Android 多平台，双线路冗余架构，本地代理自动切换。

## 架构

```
┌──────────────────────────────────────────────┐
│              Claude Code Agent                │
│         用户提到 VPN 相关操作时                  │
│         自动加载 upup-vpn-skill               │
└──────────────────┬───────────────────────────┘
                   │
    ┌──────────────┼──────────────┐
    ▼              ▼              ▼
┌────────┐  ┌──────────┐  ┌──────────┐
│ macOS  │  │ iPhone/  │  │ Android  │
│ ss-local│  │ iPad     │  │ SS App  │
└────┬───┘  └────┬─────┘  └────┬─────┘
     │           │              │
     └───────────┼──────────────┘
                 │ SOCKS5 / IKEv2
                 ▼
    ┌────────────────────────┐
    │  DigitalOcean SFO3     │
    │  Ubuntu 22.04 LTS      │
    │  ┌──────────────────┐  │
    │  │ Shadowsocks:8388 │  │  ← 主线路
    │  │ IPsec/L2TP+IKEv2 │  │  ← 备用
    │  └──────────────────┘  │
    └────────────────────────┘
```

## 功能特性 🎯

- **Claude Code 集成** — 在 Claude Code 中提到 VPN 即自动加载技能
- **双线路冗余** — Shadowsocks 主线路 + IKEv2 备用线路
- **多平台支持** — macOS 菜单栏开关 / iPhone Shadowrocket / Android SS 客户端
- **本地代理自动切换** — VPN 连断时代理环境变量自动设置/清除，零手动操作
- **一键运维** — 服务状态检查、重启、防火墙管理
- **新设备配置** — 自动生成客户端配置文件

## 双线路架构

| 线路 | 协议 | 端口 | 用途 | 优势 |
|------|------|------|------|------|
| **主线路** | Shadowsocks | 8388 | SOCKS5 代理 | 轻量、易配置、全平台支持 |
| **备用线路** | IPsec/L2TP + IKEv2 | 500/4500 | 全隧道 VPN | 原生系统支持、稳定性好 |

## 快速开始 🚀

### macOS（推荐）

桌面已有 `US VPN.app`，点击即可连接/断开。

或终端启动：
```bash
/opt/homebrew/opt/shadowsocks-libev/bin/ss-local -c /opt/homebrew/etc/shadowsocks-libev.json
```

### iPhone / iPad

1. App Store 下载 **Shadowrocket**（付费）或 **Outline**（免费）
2. 扫描 `ss://` 二维码或手动输入配置
3. 开启连接

### Android

1. 下载 [Shadowsocks for Android](https://github.com/shadowsocks/shadowsocks-android)
2. 扫描二维码或手动配置
3. 开启连接

### IKEv2 通用配置

服务器上已有导出的配置文件：
```
/root/vpnclient.mobileconfig    # iOS/macOS
/root/vpnclient.sswan           # Android StrongSwan
/root/vpnclient.p12             # 证书文件
```

重新导出：
```bash
ssh root@server "sudo ikev2.sh --exportclient <name>"
```

## 安装部署 📥

### ① 克隆到 Claude Code

```bash
cd your-project
git clone https://github.com/iTrimut/upup-vpn-skill.git .claude/commands/upup-vpn
```

### ② 验证安装

在 Claude Code 中输入 VPN 相关关键词（如"连接 VPN"），确认 Skill 自动加载。

### ③ 配置凭据

在 `references/credentials.md` 中填写服务器 IP、密码等敏感信息（此文件不入库）。

## 本地代理自动切换 ⚡

`~/.zshrc` 中配置了 zsh `precmd` hook，VPN 连断时代理环境变量自动切换：

| 状态 | 行为 |
|------|------|
| VPN 连接中（端口 1080 通） | 自动设置 `ALL_PROXY`、`HTTPS_PROXY`、`NO_PROXY` |
| VPN 断开（端口 1080 不通） | 自动清除所有代理变量 |

国内域名直连：`*.xiaomi.com`、`*.qq.com`、`*.wechat.com` 等。

## 常用运维命令

### 检查服务状态
```bash
ssh -i ~/.ssh/id_ed25519 root@<SERVER_IP> \
  "systemctl status ipsec shadowsocks-libev --no-pager | grep Active"
```

### 重启服务
```bash
ssh -i ~/.ssh/id_ed25519 root@<SERVER_IP> \
  "systemctl restart ipsec shadowsocks-libev"
```

### 防火墙规则

| 端口 | 协议 | 用途 |
|------|------|------|
| 22 | TCP | SSH |
| 500 | UDP | IKEv2 |
| 4500 | UDP | NAT-T |
| 8388 | TCP/UDP | Shadowsocks |

## Skill 触发词

```
VPN, 连接, 断开, Shadowsocks, SS, 代理, proxy,
翻墙, 科学上网, DigitalOcean, 服务器, IP 地址,
ss-local, IKEv2, L2TP, 网络工具
```

## 常见问题 🤔

❓ Q: macOS 26.5 安装 mobileconfig 后 VPN 服务不显示？
A: 使用 Shadowsocks 客户端替代，或检查系统偏好设置 → VPN。

❓ Q: AppleScript .app 提示"未验证的开发者"？
A: 右键点击 → 打开，或系统设置 → 隐私与安全性 → 仍要打开。

❓ Q: ss-local 断连怎么办？
A: 重启 ss-local 或使用脚本重连：`/Users/xiaan/Documents/VPNTool/ss_vpn.sh`

## 文件结构

```
upup-vpn-skill/
├── README.md                    # 项目说明（本文件）
├── SKILL.md                     # Claude Code Skill 主文件
├── UPDATE-2026-06-01.md         # 更新记录
└── references/
    └── credentials.md           # 服务器配置（敏感信息不入库）
```

## 反馈建议 📢

欢迎提交 [Issue](https://github.com/iTrimut/upup-vpn-skill/issues) 和 [Pull Request](https://github.com/iTrimut/upup-vpn-skill/pulls)。

## 许可证 📝

[MIT License](LICENSE)

---

<div align="center">

**私人 VPN，安全上网**

</div>
