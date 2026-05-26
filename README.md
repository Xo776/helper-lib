# YouTube Enhance for Quantumult X

YouTube & YouTube Music 去广告 + 画中画 + 后台播放 + 字幕翻译模块。

> 基于 [Maasea/sgmodule](https://github.com/Maasea/sgmodule) 原版脚本，针对 Quantumult X 做了适配修复，**完全自托管**（不依赖外部热链接）。

## 功能

| 功能 | 说明 |
|------|------|
| 🚫 去广告 | 视频贴片广告、瀑布流广告、搜索广告、Shorts 广告 |
| 🖼️ 画中画 | 强制启用 PIP（无需 Premium） |
| 🎧 后台播放 | 强制启用后台播放（无需 Premium） |
| 🌐 字幕翻译 | 自动注入 Google 翻译字幕轨道 |
| 🧭 导航栏精简 | 可屏蔽 Shorts 按钮、上传按钮、沉浸模式 |
| 📥 下载 | 强制启用视频下载选项 |

## 为什么这个版本能工作？

与失效的 `ddgksf2013/YoutubeAds.conf` 相比，本版本修复了以下致命问题：

| 问题 | ddgksf2013 | 本版本 |
|------|-----------|--------|
| QUIC 穿透 MITM | ❌ 无 UDP 阻断，广告走 QUIC 绕过 | ✅ 强制 `udp_drop_list=443` |
| protobuf 二进制损坏 | ❌ 缺失 binary-body-mode 参数 | ✅ QX 原生支持 bodyBytes |
| 脚本依赖 | ❌ 热链接外部仓库，版本不可控 | ✅ 脚本自托管在你的私有仓库 |
| googlevideo 规则 | ❌ `ctier=L` 正则已过时 | ✅ 保留兼容 + 精确 reject |

## 安装步骤

### 1. 上传到你的 GitHub 私有仓库

将整个 `by-yt` 文件夹推送到 `https://github.com/Xo776/by-yt`（仓库需设为**私有**）。

### 2. Quantumult X 配置

#### 2.1 [general] 添加 UDP 阻断（最关键！）

```
[general]
udp_drop_list=443
```

> ⚠️ **不添加这行，去广告大概率无效。** YouTube 的 QUIC (HTTP/3) 连接走 UDP 443，无法被 MITM 解密。阻断 UDP 迫使降级到 HTTPS/TCP 才能被脚本拦截。

#### 2.2 [rewrite_local] 引用

```
[rewrite_local]
https://raw.githubusercontent.com/Xo776/by-yt/main/YouTube_Enhance.conf, tag=YouTube Enhance, enabled=true
```

或者在 Quantumult X 编辑界面中直接粘贴 `YouTube_Enhance.conf` 的内容到 `[rewrite_local]` 段落下。

#### 2.3 [mitm] 引用

```
[mitm]
hostname = *.googlevideo.com, youtubei.googleapis.com, www.youtube.com, s.youtube.com
```

#### 2.4 信任证书

确保 Quantumult X 的 MITM 证书已被信任（设置 → HTTPS 解密 → 安装并信任证书）。

### 3. 验证

- 打开 YouTube App，播放视频应无贴片广告
- 切到后台应继续播放（后台播放生效）
- 全屏播放时可点击画中画按钮

## 项目结构

```
by-yt/
├── YouTube_Enhance.conf    # Quantumult X 重写配置
├── youtube.response.js     # 核心脚本 (protobuf 解析+篡改)
└── README.md               # 本文件
```

## 技术原理摘要

核心脚本通过 MITM 拦截 YouTube API 的 protobuf 二进制响应：

1. **fromBinary()**：将 `Uint8Array` 反序列化为 JS 对象
2. **去广告**：通过 "pagead" 字节特征码 + 未知字段编号 + EML 模板名 三维检测
3. **画中画/后播**：直接篡改 `playabilityStatus` 中的布尔字段
4. **字幕翻译**：克隆原始字幕轨道，追加 `&tlang=` 参数
5. **toBinary()**：重新序列化为 `Uint8Array` 返回给客户端

## 更新日志

- **2026-05-26**：初始版本，从 Maasea sgmodule 提取并适配 QX，自托管到私有仓库

## 鸣谢

- 原版脚本：[Maasea/sgmodule](https://github.com/Maasea/sgmodule)
- QX 重写参考：[ddgksf2013/Rewrite](https://github.com/ddgksf2013/Rewrite)
