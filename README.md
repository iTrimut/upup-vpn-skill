<div align="center">

# upup-vpn-skill

**一键管理你的私人 VPN 基础设施**

[![Platform](https://img.shields.io/badge/Platform-macOS%20|%20iOS%20|%20Android-lightgrey.svg)]()
[![Protocol](https://img.shields.io/badge/Protocol-Shadowsocks%20|%20IKEv2-blue.svg)]()
[![Server](https://img.shields.io/badge/Server-DigitalOcean%20SFO3-green.svg)]()
[![License](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![GitHub Stars](https://img.shields.io/github/stars/iTrimut/upup-vpn-skill?style=social)](https://github.com/iTrimut/upup-vpn-skill)

</div>

---

## 项目简介

upup-vpn-skill 是一个 **Claude Code Skill**，用于管理个人 VPN 基础设施。当用户在 Claude Code 中提到 VPN 相关操作时，自动加载此技能并执行对应任务。

支持的操作：

- VPN 连接/断开（Mac、iPhone、Android）
- VPN 服务器状态检查、迁移、故障排除
- 新设备 VPN 配置
- Shadowsocks 客户端/服务端管理
- DigitalOcean 服务器管理（创建/删除/迁移）
- IP 地址位置查询和切换

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

## 双线路架构

| 线路 | 协议 | 端口 | 用途 | 优势 |
|------|------|------|------|------|
| **主线路** | Shadowsocks | 8388 | SOCKS5 代理 | 轻量、易配置、全平台支持 |
| **备用线路** | IPsec/L2TP + IKEv2 | 500/4500 | 全隧道 VPN | 原生系统支持、稳定性好 |

## 客户端配置

### macOS

**方式一：菜单栏快捷开关（推荐）**

桌面已有 `US VPN.app`，点击即可连接/断开。

**方式二：终端命令**

```bash
# 启动
/opt/homebrew/opt/shadowsocks-libev/bin/ss-local -c /opt/homebrew/etc/shadowsocks-libev.json

# 或使用脚本
/Users/xiaan/Documents/VPNTool/ss_vpn.sh
```

**方式三：IKEv2 原生 VPN**

系统偏好设置 → VPN → 添加 VPN 配置 → IKEv2

### iPhone / iPad

推荐使用 **Shadowrocket**（付费）或 **Outline**（免费）：

1. App Store 下载 Shadowrocket 或 Outline
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

### 查看防火墙规则

```bash
ssh -i ~/.ssh/id_ed25519 root@<SERVER_IP> "ufw status numbered"
```

| 端口 | 协议 | 用途 |
|------|------|------|
| 22 | TCP | SSH |
| 500 | UDP | IKEv2 |
| 4500 | UDP | NAT-T |
| 8388 | TCP/UDP | Shadowsocks |

## Skill 触发词

当用户提到以下关键词时，Claude Code 会自动加载此 Skill：

```
VPN, 连接, 断开, Shadowsocks, SS, 代理, proxy,
翻墙, 科学上网, DigitalOcean, 服务器, IP 地址,
ss-local, IKEv2, L2TP, 网络工具
```

## 已知问题

| 问题 | 说明 | 解决方案 |
|------|------|---------|
| macOS 26.5 mobileconfig | 安装后可能不会创建 `scutil --nc` 服务 | 使用 Shadowsocks 客户端替代 |
| AppleScript .app 提示未验证 | 首次启动系统警告 | 右键打开或系统设置允许 |
| ss-local 断连 | 网络切换时可能断开 | 重启 ss-local 或使用脚本重连 |

## 文件结构

```
upup-vpn-skill/
├── README.md                    # 项目说明（本文件）
├── SKILL.md                     # Claude Code Skill 主文件
└── references/
    └── credentials.md           # 服务器配置（敏感信息不入库）
```

## 使用方式

### 作为 Claude Code Skill

此项目设计为 Claude Code 的 Skill 插件。将仓库克隆到 `.claude/commands/` 目录即可：

```bash
cd your-project
git clone https://github.com/iTrimut/upup-vpn-skill.git .claude/commands/upup-vpn
```

之后在 Claude Code 中提到 VPN 相关操作，会自动加载此 Skill。

### 直接使用

也可以直接参考 `SKILL.md` 中的配置步骤，在任意终端完成 VPN 配置。

## 许可证

[MIT License](LICENSE)

---

<div align="center">
<b>私人 VPN，安全上网</b>
</div>
