# Codex CLI 安装与配置（Windows 实记）

> 记录时间：2026-11-05　机器：Windows + pwsh 7.6.9 + Git 2.56
> 结论先说：**Codex CLI 装成功了、能跑**；**ChatGPT 桌面版装不了**（网络阻断，非技术问题）。

## 一、最终装上的是什么

| 项目 | 版本 | 位置 |
|---|---|---|
| Codex CLI | `codex-cli 0.166.0` | `D:\npm-global\node_modules\@openai\codex` |
| 启动命令 | `codex` / `codex.cmd` / `codex.ps1` | `D:\npm-global\` |
| Node.js（它依赖的运行环境） | v26.13.0 (LTS "Lithium") | `D:\nodejs\node-v26.13.0-win-x64` |
| npm | 11.20.0 | 同上 |
| 用户 PATH 新增 | `D:\nodejs\node-v26.13.0-win-x64`、`D:\npm-global` | 用户环境变量 |
| npm 全局前缀 | `D:\npm-global` | `~/.npmrc` |

验证（**新开**一个终端）：

```powershell
node -v      # v26.13.0
npm -v       # 11.20.0
codex --version   # codex-cli 0.166.0
```

## 二、为什么不是 `winget install` 或官网下载

| 路径 | 结果 |
|---|---|
| `winget install OpenAI.Codex` | 官方**没有发布 winget 包**；搜出来的 `AIChat`、`sonnylab.chatgpt`、"Codex 限速器" 都是第三方，别装 |
| `chatgpt.com/download` | ❌ 连接 0.06 秒直接失败（被阻断） |
| 微软商店（`--source msstore`） | ❌ `0x8a15005e` 服务器证书不匹配，源本身就坏了 |
| `npm install -g @openai/codex` | ✅ 14 秒装完（npm 源国内通畅） |

所以走的是 **Node → npm** 这条路。Node 不用官方安装包（慢），改从 `registry.npmmirror.com` 下 zip 解压，免管理员权限：

```powershell
curl -L -o node.zip https://registry.npmmirror.com/-/binary/node/v26.13.0/node-v26.13.0-win-x64.zip
# 解压到 D:\nodejs，再把 D:\nodejs\node-v26.13.0-win-x64 加进 PATH
npm config set prefix "D:\npm-global"      # 全局包装到独立目录，不污染其他
npm install -g @openai/codex
```

## 三、安装后踩的四个坑（都是真实报错）

### 坑 1：`npm.ps1` / `codex.ps1` 无法加载

```
无法加载文件 D:\npm-global\codex.ps1，因为在此系统上禁止运行脚本
```

原因：Windows 默认执行策略是 `Restricted`，**任何 .ps1 都不让跑**。npm 装出来的命令在 PowerShell 里是 `.ps1` 包装脚本，全被拦下。

```powershell
Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned -Force
```

`RemoteSigned` = 本地自己写的脚本可以跑、从网上下载的必须有签名。这是开发机的合理设置，只影响当前用户，可随时改回。

### 坑 2：`--profile` 不能再用旧写法的 profile 表

```
--profile `deepseek` cannot be used while config.toml contains legacy `profile = ...`
或 `[profiles.deepseek]` config; move those settings into deepseek.config.toml
```

0.166 起 profile **拆成独立文件**：`~/.codex/<名字>.config.toml`，`--profile deepseek` 就加载 `deepseek.config.toml`。

### 坑 3：`wire_api = "chat"` 已被移除

```
`wire_api = "chat"` is no longer supported. set `wire_api = "responses"`
```

这是个重要变化：Codex 不再支持 OpenAI 的 `chat/completions` 协议，**只走 `responses`**。影响是——那些只提供 chat/completions 兼容接口的国产模型，接进来可能不通（要在拿到真 key 后实测）。

### 坑 4：TOML 写错位置，配置被"吞掉"

```
warning: `model_providers.deepseek.sandbox_mode` is ignored
```

原因是 `sandbox_mode` 写在了 `[model_providers.deepseek]` 表头**后面**，TOML 会把它当成这个表的子键。**所有全局设置必须写在第一个表头之前。**

## 四、当前的两个 ~/.codex 配置文件

`config.toml`（主配置）：

```toml
sandbox_mode = "workspace-write"   # 全局设置必须在第一个表头之前

[model_providers.deepseek]
name = "DeepSeek"
base_url = "https://api.deepseek.com/v1"
env_key = "DEEPSEEK_API_KEY"       # 从环境变量读，不写死在文件里
wire_api = "responses"
```

`deepseek.config.toml`（profile）：

```toml
model = "deepseek-chat"
model_provider = "deepseek"
```

用法：`codex --profile deepseek "你的问题"`

## 五、登录这件事：OpenAI 走不通

| 域名 | 实测 |
|---|---|
| `chatgpt.com` | ❌ 0.06s 连接失败 |
| `api.openai.com` | ❌ 0.06s 连接失败 |
| `auth.openai.com` | ⚠️ 1.27s 返回 403（能连上但被 Cloudflare 拦） |
| `api.deepseek.com` | ✅ 0.4s 返回 401（服务器正常应答） |

`codex login status` → `Not logged in`。登录有两种方式，都需要能连 OpenAI：

```powershell
codex login                      # 浏览器 OAuth（ChatGPT 账号）
$env:OPENAI_API_KEY="sk-..." | codex login --with-api-key   # API key
```

## 六、用假密钥验证链路（这一步很关键）

不需要真 key，也能证明"配置对不对"：

```powershell
$env:DEEPSEEK_API_KEY = "sk-dummy-test-key"
codex --profile deepseek exec --skip-git-repo-check "reply with OK"
```

输出里的关键三行，说明整条链路已经打通，只差真 key：

```
model: deepseek-chat
provider: deepseek
ERROR: ... 401 Unauthorized ... url: https://api.deepseek.com/v1/responses
```

**401 是好事**：说明请求确实发到了 DeepSeek 的服务器，只是密钥是假的。如果配置错了，这里会是"连不上"而不是 401。

> 拿到真 key 后只需：`$env:DEEPSEEK_API_KEY="sk-真key"`（或写进系统环境变量），再跑一次上面的命令。

## 七、常用命令速查

| 命令 | 作用 |
|---|---|
| `codex` | 进入交互式会话 |
| `codex exec "问题"` | 一次性执行，不进入交互（适合脚本） |
| `codex --profile deepseek "问题"` | 用 DeepSeek 模型 |
| `codex login status` | 看登录状态 |
| `codex doctor` | 体检：安装、配置、登录、运行时一次查完 |
| `codex mcp` | 管理 MCP 服务器（和 WorkBuddy 的 MCP 是同一套概念） |
| `codex update` | 升级自己 |

## 八、我的理解（用自己的话写）

- **Codex 是"壳"，模型是"脑子"**：壳装在哪台机器、用什么语言写都不重要，重要的是它去敲哪个 `base_url`。改了 `base_url`，同一个壳就能换成国产脑子。
- **`env_key` 的设计值得学**：配置文件里只写变量名，密钥放环境变量。这样配置文件可以进 Git 仓库、可以发给别人，不怕泄露。
- **报错信息里藏着答案**：`wire_api = "chat" is no longer supported` 直接告诉你改成什么；`sandbox_mode is ignored` 直接告诉你是哪个键。学会读报错，比搜教程快。
