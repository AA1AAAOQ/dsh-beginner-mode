# 小白模式 · Beginner Mode

[![Release](https://img.shields.io/github/v/release/AA1AAAOQ/dsh-beginner-mode?label=release)](https://github.com/AA1AAAOQ/dsh-beginner-mode/releases/latest)
[![License](https://img.shields.io/github/license/AA1AAAOQ/dsh-beginner-mode)](LICENSE)
[![Built for dsh](https://img.shields.io/badge/dsh-0.2.2--rc.3-blue)](https://github.com/deepseek-ai/deepseek-harness)
[![Validate](https://github.com/AA1AAAOQ/dsh-beginner-mode/actions/workflows/validate.yml/badge.svg)](https://github.com/AA1AAAOQ/dsh-beginner-mode/actions/workflows/validate.yml)

给 [DeepSeek Harness](https://github.com/deepseek-ai/deepseek-harness)（dsh）用的一个 **可选** agent 预设。

装好之后，在预设菜单里多出一项 **小白模式**。选它，AI 就把用户当成几乎没有电脑知识的人：说人话、一次只教一步、不清楚就问、没把握就联网查、危险操作先如实说后果。不选它，一切和装之前一样。

> 它**不改工具、不改权限、不影响其它预设**。工具一个没加、一个没减，沙箱模式跟原来一样。

## 为什么会有这个

现在的大模型默认把用户当成工程师：一次丢十步、满屏术语、猜错需求就开始修。对真正不懂电脑的人，这套话术是有害的——他们会点错、会以为是自己笨、会在弹窗前不敢动。

这个预设只做一件事：**把说话的层次降下来**。它把「不懂」当成默认前提，而不是需要用户承认的缺陷。

## 它到底改了什么

改的是 `@deepseek-ai/dsh-persona` 这一行的 `prefix` 和 `suffix`，也就是系统的说话方式。具体规则包括：

| 方面 | 规则 |
|---|---|
| 语气 | 短句、常用词、一句一个意思；禁用 "it's easy"、"just"、"simply" 这类词 |
| 术语 | 非用不可的电脑词，当场用几个大白话解释；优先用日常词（"文件夹" 而不是 "目录"） |
| 步长 | **一条消息只给一步**，说完停下来等回话；绝不一次丢整份说明书 |
| 澄清 | 用户只打一个「卡」「打不开」时**不猜**，给两三个能选的具体选项 |
| 版本 | 按 `CurrentBuild >= 22000` 判定 Win10/Win11，**不信 ProductName**（Win11 常把它报成 "Windows 10 Pro"） |
| 事实 | 菜单路径、下载链接、版本号、价格，**先联网查**再报，并说出出处 |
| 不确定性 | WMI/CIM 被拒不等于是"版本未知"，改读注册表；读不到就如实说无法确认 |
| 屏幕 | 进程列表**为空不构成任何证据**（沙箱只看得见自己起的进程），必须问用户屏幕上看到什么 |
| 启动程序 | 返回成功 ≠ 窗口出现了；不许回报"我已经打开了"，要问用户确认 |
| 显示窗口 | 开文件夹/装程序/开浏览器属于受限窗口做不可靠的操作，**先申请完全权限**再动手 |
| 装软件 | **不走 winget / 包管理器 / 静默安装**，只从官网下载；下完先问"要我打开还是你自己双击" |
| 危险操作 | 删除、格式化、卸载杀毒等，**先如实说清后果**，用户坚持就照做，不阻拦、不说教 |
| 拒绝分类 | 文件拒绝 / 系统查询拒绝 / 网络证书失败，三类分开处理；证书错先怀疑本机代理或加速器 |
| 隐私 | 绝不要密码、卡号、短信验证码；说明"验证码是账户钥匙"，并点破电话/弹窗诈骗 |

完整规则原文就在 [`cordis.patch.yml`](cordis.patch.yml) 的 `persona` 行里（`config.suffix`），改措辞直接改那里。

## 安装

### 方式一：从 Release 安装（推荐）

1. 下载最新的 `.tgz`（在本仓库 **Releases** 页）。
2. 打开 dsh 的 **插件** 页 → **Add plugin** → 选择本地的 `.tgz` 文件。
3. 启用后**新开一个会话**，在预设菜单里选 **小白模式**。

### 方式二：从目录安装

把本仓库克隆或解压到任意本地目录，然后在 dsh 的 **插件** 页用 **Add plugin** 指向该目录。

### 方式三：给会写配置的人

装到 profile 后，`cordis.patch.yml` 会插入一行：

```yaml
- insert:
    - id: preset-beginner
      name: '@deepseek-ai/dsh-agent-preset'
      config:
        id: beginner
        name: 小白模式
        order: 1.5
        plugins: [ ... ]   # standard 预设的完整列表，仅 persona 一行不同
```

`config.plugins` **就是 `standard` 预设的完整插件列表**，只替换了 `persona` 的 `prefix`/`suffix`。原版 `standard` 不会被改。

## 使用

**已有会话不会变**——它们保持开始时选的预设。必须**新建会话**才能用上小白模式。

验证它是否生效，可以试这三件事：

1. 问一个需要多步的问题，看它是否**只给一步就停下**。
2. 只打一个字（比如「卡」），看它是否**反问**而不是猜。
3. 问某个 Windows 设置在哪，看它是否**联网查**并说明出处。

## 前置条件

- DeepSeek Harness，且版本的 `@deepseek-ai/dsh-web-app` 提供 `standard` 预设（本包按 dsh `0.2.2-rc.3` 的 `presets/standard.patch.yml` 编写）。
- 不需要任何额外依赖、构建步骤或原生模块。

> **版本兼容**：`config.plugins` 是 `standard` 预设的**快照**。dsh 升级后若 `standard` 的插件列表变化（增删工具），本预设不会自动跟随。升级 dsh 后建议照着新版 `presets/standard.patch.yml` 重新核对一遍。

## 它与沙箱的关系

这个预设**不改变权限**，但它明确写下了沙箱的真实限制，因为不写会让 AI 对用户说假话。实测（Windows 11，dsh `0.2.2-rc.3`）：

| | `workspace-write`（受限） | `danger-full-access` |
|---|---|---|
| 可见进程数 | **10** | **287** |
| 能看到 `explorer` 吗 | **不能（0）** | 能（正常计数） |
| 代启 `explorer.exe` | 弹 `0xc0000142` | 正常打开 |
| `USERPROFILE\Downloads` 可写 | **不可写** | 可写 |
| WMI (`Get-CimInstance`) | **拒绝访问** | 可用 |

所以预设会：不拿进程表当证据、开窗口前先申请权限、把 `0xc0000142` 解释成"受限窗口所致，不是你电脑坏了"。

## 许可

MIT，见 [LICENSE](LICENSE)。

## 免责

这是**提示词**，不是安全沙箱。它不阻止任何工具调用，也不扩大权限。规则能被模型忽略——如果它仍然说了不当的话，请把具体那句话贴出来，改的是 `cordis.patch.yml` 里 `persona` 行的措辞。
