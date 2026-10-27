# Xray VLESS Reality 一键安装脚本

一个基于 Python 的 **Xray-core（VLESS + Reality）一键安装脚本**，支持自动配置、随机 SNI 选择以及 IPv4 / IPv6 客户端分享链接输出。

---

## 免责声明

本项目仅供 **学习与个人研究用途**。

- 使用者需自行遵守当地法律法规  
- 作者不对任何滥用行为或损失负责  

---

## 🚀 功能特点

- 一键安装 Xray-core  
- 自动配置 VLESS + Reality 协议  
- 随机 SNI（伪装域名选择）  
- 自动生成 UUID 和 Reality 密钥  
- 自动生成 Short ID  
- 自动配置 systemd 服务
- 自动开启 Google BBR 防拥堵算法
- 自动生成客户端分享链接（VLESS URI）
- 分别检测公网 IPv4 和 IPv6，有哪个地址就输出对应链接
- IPv6 分享链接自动添加 URI 必需的方括号 `[]`

---

## 使用前准备工作

先获取一个可远程控制的VPS，推荐使用DigitalOcean、搬瓦工、Vultr

### 📦 VPS系统要求

支持系统：

- Ubuntu 20.04 / 22.04 / 24.04  
- Debian 11+  
- CentOS  

运行要求：

- Root 或 sudo 权限  
- Python 3.8+

---

## ⚡ 安装方法

### 一键安装命令

```bash
curl -fL https://raw.githubusercontent.com/change93n98/xray-vless-reality-installer/main/install.py -o install.py && sudo python3 install.py
```

以上命令下载本仓库的 `install.py`，包含 IPv4 / IPv6 分享链接修复。

> **已有安装请先阅读下面的服务管理说明。** 再次运行安装脚本会生成新的 UUID、Reality 密钥和 Short ID，并覆盖 `/usr/local/etc/xray/config.json`，原客户端节点参数将失效。仅修正分享链接或重启服务，无需重新安装。

`install-for-debian.py` 是仅使用 APT 的另一份安装脚本，也支持双栈链接输出；自动开启 BBR 和配置防火墙的流程位于上面的 `install.py` 中。

---

## 📱 输出示例

安装完成后，你将获得：

- UUID
- Reality 公钥
- Short ID
- 选定的 Reality 域名（SNI）
- IPv4 / IPv6 的 VLESS 分享链接（取决于服务器能检测到哪些公网地址）

示例：
```text
IPv4:
vless://UUID@YOUR_IPV4:443?encryption=none&flow=xtls-rprx-vision&security=reality&sni=YOUR_SNI&fp=chrome&pbk=PUBLIC_KEY&sid=SHORT_ID#Xray

IPv6:
vless://UUID@[YOUR_IPV6]:443?encryption=none&flow=xtls-rprx-vision&security=reality&sni=YOUR_SNI&fp=chrome&pbk=PUBLIC_KEY&sid=SHORT_ID#Xray
```

示例中的 `UUID`、地址、`YOUR_SNI`、`PUBLIC_KEY` 和 `SHORT_ID` 都需要使用安装脚本实际输出的值。脚本会随机选择 SNI，不要直接照抄示例域名。

### 导入 v2rayN

1. 复制完整的 `vless://` 分享链接。
2. 回到 v2rayN 主界面，按 `Ctrl + V` 从剪贴板导入。
3. 选中节点并设为活动服务器，使用支持 VLESS + Reality 的核心（例如 Xray）。

客户端没有可用 IPv6 时，请导入 **IPv4 链接**。IPv6 地址在 URI 中必须写成 `[IPv6地址]:443`；如果旧链接缺少方括号，只需修正链接，不需要重建服务端配置。两条链接使用同一组 UUID、Reality 公钥和 Short ID，仅服务器地址不同。

### 已安装服务器：检查和重启服务

先备份并校验当前配置，校验成功后再重启：

```bash
sudo cp -p /usr/local/etc/xray/config.json /usr/local/etc/xray/config.json.bak-$(date +%Y%m%d-%H%M%S)
sudo /usr/local/bin/xray run -test -config /usr/local/etc/xray/config.json && sudo systemctl restart xray
sudo systemctl status xray --no-pager -l
```

查看近期日志和 443 端口监听状态：

```bash
sudo journalctl -u xray -n 50 --no-pager
sudo ss -lntp 'sport = :443'
```

重启服务会短暂中断现有连接，但不会重新生成节点参数。服务端配置中包含 Reality 私钥，请妥善保管备份，不要将其提交到仓库或公开分享。

---
## 工作原理

该脚本主要执行以下步骤：

- 更新软件包列表  
- 使用官方安装脚本安装 Xray-core  
- 自动生成：
  - UUID（客户端 ID）
  - X25519 Reality 密钥对
  - Short ID  
- 随机选择 TLS 伪装域名（SNI）  
- 写入 Xray 配置文件：`/usr/local/etc/xray/config.json`  
- 启用 systemd 服务
- 开启谷歌BBR防拥堵算法
- 生成 VLESS 分享链接  

---

## 📄 使用许可

MIT License

---

## ⭐ 给项目点星

如果你觉得这个项目有帮助，欢迎点一个 Star ⭐
