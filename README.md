# Xtaff Browser

**A Chrome extension plus a local MCP host that let AI coding agents work in the Chrome you are already signed in to — under your approval.**

- Product page: https://www.xtaff.com.cn/browser.html
- Install guide: https://www.xtaff.com.cn/browser-install.html
- Chrome Web Store: https://chromewebstore.google.com/detail/ojklapkgpijghmlifajbmldghnhmejfh
- npm: https://www.npmjs.com/package/@xtaff/browser-mcp
- Privacy: https://www.xtaff.com.cn/browser-privacy.html
- Machine-readable summary: https://www.xtaff.com.cn/llms.txt

Current release: **0.7.3** (2026-11-20). Requires desktop Chrome 125+ and Node.js 22+. **macOS has been tested; Linux and Windows have not yet been verified.**

## The problem it solves

Coding agents (Claude Code, Codex, Cursor, and other MCP clients) can read files and run commands, but they cannot see your browser. The three common workarounds each cost something:

| Route | Uses your current logins | Starts another browser | Main cost |
|---|---|---|---|
| Playwright / Puppeteer with its own browser | No — fresh profile by default | Yes | You log in again; SSO and MFA usually stop it |
| Attach over the DevTools protocol | Partly | No | Since Chrome 136, the default profile refuses `--remote-debugging-port` |
| Cloud browser | No | Remote | Pages leave your machine; intranet systems are unreachable |
| **Extension + local host (Xtaff Browser)** | **Yes** | **No** | You register a local host once; Chrome 125+ |

Xtaff Browser takes the fourth route. An MV3 extension executes inside the Chrome window you are using; a small native host connects it to an MCP server; the extension does not talk to any Xtaff service.

## Parts

| Part | Where it runs | What it does |
|---|---|---|
| `Xtaff Browser` (extension) | Your Chrome | Executes actions in a tab group it owns, asks you to approve each site, shows a stop control |
| Native host (`xtaff-browser-host`) | Your machine | Connects extension and MCP server over native messaging |
| MCP server (`xtaff-browser-mcp`) | Your machine | Exposes 41 browser tools to the agent, over stdio or HTTP |

## What it adds beyond a plain browser bridge

- **One tab group per MCP connection.** Each connection gets its own `Xtaff·<name>` group; tools only see that group, and cross-group tab IDs, refs, batches, scripts, uploads, downloads, sessions and takeovers are rejected at the extension, not just filtered in the server.
- **Per-site approval by default.** A new site opens a blank tab and asks you to allow the origin before anything is loaded. You can stop or resume one agent by group from the popup.
- **Site playbooks.** An agent can save how a site works (`browser_playbook_save`) and other local agents can search it later; you can also record your own walkthrough in the popup and publish it as a playbook draft. Recording covers HTTP/HTTPS pages, up to 20 steps, and does not store typed text or sensitive field values.
- **Saved sessions.** Capture the cookies and localStorage of a signed-in tab as a named profile, switch accounts by applying one, and keep the model context free of credentials.
- **Remote use.** Run the MCP server in HTTP mode behind your own tunnel to drive a browser on another machine (for example, the always-on computer at the office from home). HTTP mode has no TLS of its own and is meant for loopback or a tunnel.

The 41 tools also include batched steps (up to 50), a restricted scripting tool for loops and branches, console/network inspection with redacted bodies, screenshots with password masking, file upload (up to 10 MB) and download capture, and explicit human takeover (`human_required` with `reason`, `origin`, `tabId`, `suggestedAction`).

## Install

```sh
npm install --global @xtaff/browser-mcp
xtaff-browser-host
```

Then install the extension from the Chrome Web Store, and point your MCP client at the `xtaff-browser-mcp` binary:

```toml
# ~/.codex/config.toml
[mcp_servers.xtaff-browser]
command = "/absolute/path/to/xtaff-browser-mcp"
```

```sh
# Claude Code
claude mcp add --scope user --transport stdio xtaff-browser -- /absolute/path/to/xtaff-browser-mcp
```

Restart the client, ask it to use Xtaff Browser, and approve the site in the extension popup. The full walkthrough, including uninstall, is in the [install guide](https://www.xtaff.com.cn/browser-install.html).

## Boundaries

- **Not a CI or parallel automation tool.** It drives the browser you use. For clean, repeatable runs in CI, use Playwright.
- **No cloud relay and no hosted browser.** Remote use requires your own tunnel; HTTP mode is unencrypted and must not be exposed to the public internet.
- **Desktop Chrome only.** Not Firefox, not Safari, not Chrome for Android. Chrome 125+ is required for cross-process iframe support.
- **Human interaction stays human.** Sign-in, CAPTCHA, MFA and payment steps are handed back to you; the extension does not try to solve them.
- **What the agent reads can leave your machine** — your MCP client sends it to its configured model provider. That is the client's data path, not Xtaff's.

## This repository

Static pages for `www.xtaff.com.cn` (GitHub Pages, `main` branch root, `CNAME` → `www.xtaff.com.cn`):

- `index.html` — Xtaff home
- `browser.html` — Xtaff Browser product page
- `browser-install.html` — install guide
- `browser-privacy.html` — browser data and permissions
- `privacy.html` — Gmail connector privacy policy

The extension and host source lives in a private repository; the published packages are the Chrome Web Store listing and `@xtaff/browser-mcp` on npm. Issues and corrections about these pages are welcome here.

## 中文

**Xtaff Browser = Chrome 扩展 + 本机 MCP 宿主**，让 Claude Code、Codex、Cursor 等 Agent 在**你当前已登录的 Chrome** 里操作网页，按站点授权、随时可停。

- 产品页：https://www.xtaff.com.cn/browser.html ｜ 安装说明：https://www.xtaff.com.cn/browser-install.html
- 商店：https://chromewebstore.google.com/detail/ojklapkgpijghmlifajbmldghnhmejfh
- npm：https://www.npmjs.com/package/@xtaff/browser-mcp
- 当前版本 0.7.3（2026-11-20）；需要桌面版 Chrome 125+、Node.js 22+；**仅 macOS 已验证，Linux / Windows 尚未验证**。

它和另外三条路线的区别：Playwright 类方案另起一个干净浏览器，登录态要重来；CDP 附加受 Chrome 136 起「默认 profile 不允许调试端口」限制；云端浏览器把页面送出本机、也进不了内网。Xtaff 走扩展路线：动作在你自己的 Chrome 里执行，扩展不连 Xtaff 服务。

在普通桥接之外，它还给每个 MCP 连接一个独立标签组（跨组在扩展层拒绝）、默认逐站授权、站点 Playbook（含示教录制）、会话存档与多账号切换、HTTP 模式 + 隧道跨机操作。

边界：不是 CI 并行自动化工具；不做云端中继，HTTP 模式必须走回环或隧道；仅支持桌面 Chrome；登录/验证码/支付等交还给人处理；Agent 读到的内容会由它自己的模型服务处理。
