# DeepSeek API 接入实记

> 日期：2026-11-05　余额：充值 1 元，实测消耗 0.02 元
> 结论：**密钥可用，Codex 已经能拿 DeepSeek 的模型在本地真干活**（写文件 + 跑脚本）。

## 一、密钥放在哪里（重要）

密钥**没有写进任何文件**，只写进了用户环境变量：

```powershell
[Environment]::SetEnvironmentVariable('DEEPSEEK_API_KEY', 'sk-...', 'User')
```

为什么要这样：

| 做法 | 风险 |
|---|---|
| 写进代码 / 配置文件 | 一旦提交到 Git、发给别人、截图，就泄露了 |
| 写进环境变量 | 只存在于本机用户注册表，配置文件里只留变量名 `env_key = "DEEPSEEK_API_KEY"` |

配置里那句 `env_key` 就是干这个的：**配置文件可以公开，密钥不行**。

> 补充：这个 key 现在也出现在聊天记录里了。练手无妨（余额只有 1 元），但**好习惯是事后去控制台删除重建**。

## 二、先摸清接口的脾气

### 1. 密钥是否有效 + 余额

```powershell
$h = @{ 'Authorization' = "Bearer $env:DEEPSEEK_API_KEY" }
Invoke-RestMethod 'https://api.deepseek.com/user/balance' -Headers $h
```

返回：

```json
{ "is_available": true,
  "balance_infos": [ { "currency": "CNY", "total_balance": "1.00",
                       "granted_balance": "0.00", "topped_up_balance": "1.00" } ] }
```

`is_available: true` = 密钥有效且有钱。这一句比什么都有用，出问题先查它。

### 2. 模型名（和网上老教程不一样！）

```powershell
Invoke-RestMethod 'https://api.deepseek.com/models' -Headers $h
```

实际返回只有两个：

| 模型 id | 定位 |
|---|---|
| `deepseek-flash` | 快、便宜，日常够用 |
| `deepseek-v4-pro` | 更强，慢一点贵一点，适合难题 |

**注意**：网上 90% 的教程还写着 `deepseek-chat` / `deepseek-reasoner`，那些名字现在已经不在列表里了。**以 `/models` 返回为准**，不要照抄教程。

### 3. 两种协议都支持

| 端点 | 用途 | 实测 |
|---|---|---|
| `POST /chat/completions` | OpenAI 传统协议，绝大多数客户端用它 | ✅ 通 |
| `POST /v1/responses` | OpenAI **新版** Responses 协议，Codex 0.166 只认这个 | ✅ 通 |

这一点很关键：上一节笔记里说 Codex 0.166 "只支持 responses、chat 已被移除"，我当时担心 DeepSeek 接不上。实测 DeepSeek **已经支持 responses**，所以 Codex 能直接用。

## 三、Codex 的两个 profile

`~/.codex/deepseek.config.toml`：

```toml
model = "deepseek-flash"
model_provider = "deepseek"
```

`~/.codex/deepseek-pro.config.toml`：

```toml
model = "deepseek-v4-pro"
model_provider = "deepseek"
```

用法：

```powershell
codex --profile deepseek "你的问题"         # flash，便宜快
codex --profile deepseek-pro "你的问题"     # pro，难题用
```

## 四、真实跑一次任务（这一步才叫"接入成功"）

目标：让它建一个文件、跑起来、把结果给我。

```powershell
cd D:\codex-demo
codex --profile deepseek exec --skip-git-repo-check --dangerously-bypass-approvals-and-sandbox `
  "Create hello.py: print 'DeepSeek + Codex OK', then the sum of 1..100 from a loop, then today's date. Then run it with python and show the output."
```

它自己做了三件事，输出如下：

```
A hello.py
exec "D:\PowerShell7\7.6.9\pwsh.exe" -Command 'python hello.py' in D:\codex-demo
succeeded in 6746ms:
DeepSeek + Codex OK
5050
2026-11-05
tokens used 3,928
```

生成的 `hello.py`：

```python
from datetime import date

print("DeepSeek + Codex OK")

total = 0
for number in range(1, 101):
    total += number
print(total)

print(date.today())
```

注意细节：它调用的 shell 是 **pwsh 7**（`D:\PowerShell7\7.6.9\pwsh.exe`），不是老的 PowerShell 5.1 —— 因为我昨天把默认 shell 改了，这里就看出了效果。

**成本**：前后跑了 4 次（含失败重试），余额从 1.00 → 0.98，共 **0.02 元**。

## 五、踩的坑：Windows 上沙箱不生效

传 `-s workspace-write` 也没用，它仍然报：

```
sandbox: read-only
exec_command tool returns Rejected(... blocked by policy)
```

原因是 **Codex 的沙箱机制（landlock / seatbelt）是 Linux 和 macOS 的特性，Windows 上没实现，直接退回只读**。于是模型能"想"、不能"做"——它会老老实实告诉你"文件没创建"，而不是假装成功（这一点比很多工具强）。

两种解法：

| 参数 | 效果 | 什么时候用 |
|---|---|---|
| `--dangerously-bypass-approvals-and-sandbox` | 完全不沙箱、不询问，直接执行 | 只在专门的空目录里做实验 |
| `-s read-only`（默认） | 只读，光看不改 | 让它读代码、解释、review |

**风险提示**：bootstrap 参数等于把终端交给模型。**永远不要在 `C:\Windows`、用户主目录、有其他重要文件的目录里用它**。实验就新建一个空目录（我建的是 `D:\codex-demo`）。

## 六、我的理解

- **"接入 API"成功与否，只取决于两件事**：密钥有权限、端点说同一种协议。跟客户端长什么样无关。
- **模型名要现查，不能抄教程**。`/models` 接口就是权威答案。这次要不是查了，我会一直用 `deepseek-chat` 然后收到 404。
- **沙箱没生效 ≠ 工具坏了，是平台没实现**。学会看它到底卡在哪一层（是工具拒绝、还是策略拒绝、还是网络拒绝），比反复重试有效。
- **0.02 元跑 4 次**：API 计费按 token，日常练手 1 块钱能用很久。真正贵的是用大模型跑长上下文。
