# pi-obscura-skill

[简体中文](./README.zh-CN.md) | [English](./README.en.md) | [日本語](./README.ja.md)

用 [Obscura](https://github.com/h4ckf0r0day/obscura) 作为 Pi 的轻量级无头浏览器，而不是驱动一个完整的 Chrome。

Pi 中的 skill 名称：`obscura-cdp`

## 产品定位

- `pi-obscura-skill`：轻量、低内存、默认首选
- `pi-browser-hybrid-skill`：先探测兼容性，不稳就自动切到 Chrome 路线

## 为什么做这个包

这个包可以看作 `chrome-cdp-skill` 的 Obscura 版本，但不是简单照搬，而是按 Obscura 的真实能力重新设计。

主要特点：

- 相比 headless Chrome 内存占用更低
- 按需自动启动本地 Obscura daemon
- 不需要 per-tab keepalive daemon
- 重点使用 Obscura 目前稳定的能力：Markdown 快照、HTML 检查、JS 执行、选择器点击、表单填写、页面跳转、原始 CDP 调用
- 不走截图优先的工作流，因为当前 Obscura 版本还没有实现 `Page.captureScreenshot`

## 安装

### 推 GitHub 之前，直接本地安装

```bash
pi install /Users/daidai/ai/pi-obscura-skill
```

### 推到 GitHub 之后安装

```bash
pi install git:github.com/daidai118/pi-obscura-skill
```

### 安装到当前项目的 Pi 配置，而不是全局配置

```bash
pi install -l git:github.com/daidai118/pi-obscura-skill
```

## 后续更新

只更新这个包：

```bash
pi update git:github.com/daidai118/pi-obscura-skill
```

更新所有未固定版本的 Pi 扩展包：

```bash
pi update --extensions
```

如果你希望以后更新方便，安装时**不要**固定 git ref。

便于后续更新：

```bash
pi install git:github.com/daidai118/pi-obscura-skill
```

固定版本后，`pi update` 会跳过它：

```bash
pi install git:github.com/daidai118/pi-obscura-skill@v0.1.1
```

## 项目文档

- [Contributing guide](./CONTRIBUTING.md)
- [Roadmap](./ROADMAP.md)
- [Release checklist](./RELEASE_CHECKLIST.md)

## 运行要求

- Node.js 22+
- 支持自动下载的 Obscura 发布二进制，或者你自己通过 `OBSCURA_BIN` 指定本地 Obscura

如果系统 `PATH` 里已经有 `obscura`，skill 会直接使用它。
否则会自动下载已验证的版本。

## 站点兼容性自检

在自动化一个新站点前，先跑：

```bash
skills/obscura-cdp/scripts/obscura-cdp.mjs check https://example.com
skills/obscura-cdp/scripts/obscura-cdp.mjs check --json https://example.com
```

返回状态含义：

- `compatible` → 直接用 Obscura
- `risky` → 更建议走 Chrome fallback
- `incompatible` → 直接改走 hybrid / Chrome

这个自检命令是这次专门补进去的，因为我本地验证发现：有些站点 Obscura 能正常跑，但像 `https://100t.xiaomimimo.com/` 这种站点会出现样式未真正应用、布局塌缩、交互流程点不出来等问题。

## 命令

所有命令都通过下面这个脚本执行：

```bash
skills/obscura-cdp/scripts/obscura-cdp.mjs
```

### 生命周期

```bash
skills/obscura-cdp/scripts/obscura-cdp.mjs start
skills/obscura-cdp/scripts/obscura-cdp.mjs status
skills/obscura-cdp/scripts/obscura-cdp.mjs stop
```

### 页面管理

```bash
skills/obscura-cdp/scripts/obscura-cdp.mjs list
skills/obscura-cdp/scripts/obscura-cdp.mjs open https://example.com
skills/obscura-cdp/scripts/obscura-cdp.mjs close <target>
skills/obscura-cdp/scripts/obscura-cdp.mjs nav <target> https://example.com
```

### 页面读取

```bash
skills/obscura-cdp/scripts/obscura-cdp.mjs md   <target>
skills/obscura-cdp/scripts/obscura-cdp.mjs snap <target>
skills/obscura-cdp/scripts/obscura-cdp.mjs html <target> [selector]
skills/obscura-cdp/scripts/obscura-cdp.mjs eval <target> "document.title"
skills/obscura-cdp/scripts/obscura-cdp.mjs net  <target>
skills/obscura-cdp/scripts/obscura-cdp.mjs evalraw <target> "DOM.getDocument" '{}'
```

`md` / `snap` 会使用 Obscura 的 `LP.getMarkdown` 返回 Markdown 快照。

### 页面交互

```bash
skills/obscura-cdp/scripts/obscura-cdp.mjs click <target> "button.submit"
skills/obscura-cdp/scripts/obscura-cdp.mjs fill  <target> "input[name=q]" "pi agent"
skills/obscura-cdp/scripts/obscura-cdp.mjs type  <target> "hello world"
skills/obscura-cdp/scripts/obscura-cdp.mjs loadall <target> ".load-more" [ms]
```

如果你有稳定选择器，优先用 `fill`，比 `type` 更稳。

## 环境变量

```bash
OBSCURA_BIN=/path/to/obscura
OBSCURA_PORT=9223
OBSCURA_STEALTH=1
OBSCURA_WORKERS=4
OBSCURA_PROXY=http://127.0.0.1:8080
OBSCURA_AUTO_INSTALL=0
OBSCURA_VERSION=v0.1.1
OBSCURA_CHECK_SETTLE_MS=1500
```

## 设计说明

### 为什么默认端口是 9223？

Obscura 自己默认是 `9222`，但 `9222` 也是 Chrome 远程调试最常见的端口。这里默认改成 `9223`，这样迁移时可以并存。

### 为什么没有截图命令？

因为当前 Obscura 版本没有实现 `Page.captureScreenshot`。如果硬做会很不稳定，所以目前主路线是 Markdown 和 HTML 检查。

### 为什么不需要 per-tab daemon？

Chrome 版之所以需要，是为了避免反复弹出调试授权，并保持 tab 会话常驻。Obscura 本身已经是本地 headless daemon，所以这里每次命令只需要重新连接 browser websocket 并重新 attach target 即可。
