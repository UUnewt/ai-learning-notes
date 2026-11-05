# AI 学习笔记

记录我在 AI 工具链上的学习过程：术语概念、环境配置、踩坑记录。

## 目录

| 文件 | 内容 | 完成日期 |
|---|---|---|
| [`notes/01-术语与生态地图.md`](notes/01-术语与生态地图.md) | LLM / Model / Harness / Agent 的定义与关系，11 个常见名字的分类 | 2026-11-04 |
| [`notes/02-Markdown语法速查.md`](notes/02-Markdown语法速查.md) | 常用 Markdown 语法与我实际会用的部分 | 2026-11-04 |
| [`notes/03-Git与GitHub速查.md`](notes/03-Git与GitHub速查.md) | Git 四个区、四条命令、SSH 配置、踩过的坑 | 2026-11-04 |
| [`notes/04-Codex安装与配置.md`](notes/04-Codex安装与配置.md) | Codex CLI 0.166 的安装路径、四个真实报错、DeepSeek 入口配置与链路验证 | 2026-11-05 |

## 我的学习路线

- [x] 术语地图（LLM / Model / Harness / Agent）
- [x] Markdown 写作 + VS Code 扩展
- [x] shell / 终端 / pwsh 7
- [x] GitHub 账号与 Git 四命令 → 仓库已上线：<https://github.com/UUnewt/ai-learning-notes>
- [x] Codex CLI 安装（ChatGPT 桌面版因网络阻断无法安装）
- [ ] Skills（Humanizer-zh）
- [ ] MCP（GitHub MCP Server）
- [ ] DeepSeek API 接入（Codex 的接入链路已用假 key 验证通过）

## 本地环境

- OS：Windows
- 编辑器：VS Code（扩展：Markdown All in One、Markdown Preview Enhanced、markdownlint）
- 版本管理：Git 2.56.0.4（安装在 `D:\Git`）
- 终端：pwsh 7.6.9（`D:\PowerShell7\7.6.9`）
- 运行时：Node.js v26.13.0 + npm 11.20.0（`D:\nodejs\node-v26.13.0-win-x64`，全局包在 `D:\npm-global`）
- AI CLI：Codex CLI 0.166.0（`codex --profile deepseek` 走 DeepSeek）

---

> 这个仓库本身就是第 8 项任务的材料：用 `git init` 管理，再 push 到 GitHub。
