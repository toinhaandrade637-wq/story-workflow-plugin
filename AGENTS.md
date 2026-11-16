# AGENTS.md — 给 AI 助手的接入手册

> 你（AI 助手）被要求帮用户安装或使用这个仓库。**先读完这一页再动手。**
> 这一页只讲「怎么接对、怎么不出错」，不讲这个软件能干什么（那是 README 的事）。

---

## 0. 三十秒判断：这个用户的客户端能不能用

**能用的前提只有一个：宿主敢在本机起一个 `node` 进程。**

本仓库同时提供两种接入方式，覆盖两类宿主：

**A. 支持 Agent Plugins 1.0.0 的宿主** —— 把仓库目录整个交给它即可，不用手写配置。
本仓库根有标准 `plugin.json` + `mcp.json`，已通过官方 schema 校验。
这一类包括 ChatGPT 桌面端、Codex、Cursor、GitHub Copilot、VS Code、Kiro 等支持该标准的客户端。

**B. 只支持 MCP、不认 Agent Plugins 的宿主** —— 用第 4 节那段 JSON 手工接。
这一类包括 Claude Desktop、Cline、Roo Code、Zed、Continue、WorkRally。

**不能用的：纯网页聊天。** ChatGPT 网页版、Claude.ai 网页版，以及任何只能发 HTTP 请求、
不能在本机起进程的对话产品。它们只接受 **HTTPS 远程 MCP**。

> **不要**让用户拿网页产品「试试看能不能连」。连不上不是配置问题，是架构上不可能。
> 用户坚持要用网页版，就明确说：需要有人把这个服务部署成远程端点，本仓库不含这个能力。
>
> 注意区分：**ChatGPT 桌面端**（装插件）和 **ChatGPT 网页版**（对话）是两回事。
> 前者走 Agent Plugins 有机会用，后者不行。别混为一谈。

---

## 1. 安装前必须先确认 Node

```bash
node -v
```

**要求 v20.6.0 或更高。** 低于这个版本，服务端起不来。

- 没有 Node 或版本低 → 让用户先装/升级 Node LTS（<https://nodejs.org>），**不要**尝试替他装。
- 用户说「装了但还是找不到命令」→ 让他在**同一个终端**里重开一次；仍不行就是 PATH 没配，让他用 Node 的绝对路径。

**服务端已经打包成单文件，不需要 `npm install`，不需要编译，不需要 `tsx`。**
如果用户问「要不要先 install」，回答：不需要。

---

## 2. 拿到代码

```bash
git clone https://github.com/toinhaandrade637-wq/story-workflow-plugin.git
```

把得到的**绝对路径**记下来。下面所有配置都要用它。

**不要**把仓库 clone 到临时目录、下载文件夹、或任何会被清理的地方。

---

## 3. 先自检服务端，再接客户端

这一步能提前发现 90% 的问题，**不要跳过**：

```bash
node <仓库绝对路径>/server/index.mjs
```

| 现象 | 含义 | 处置 |
|---|---|---|
| 卡住不动、没有任何输出 | **正常**。它在等 stdin 的 MCP 消息 | `Ctrl+C` 退出，继续第 4 步 |
| `Cannot find module` | 路径错 | 检查是不是指向 `server/index.mjs` |
| 语法错误 / `Unexpected token` | Node 版本太低 | 回到第 1 步 |
| 立刻退出并打印一堆东西 | 包不完整 | 重新 clone |

---

## 4. 写配置

配置形状（几乎所有客户端都认这个结构）：

```json
{
  "mcpServers": {
    "story-workflow": {
      "command": "node",
      "args": ["<仓库绝对路径>/server/index.mjs"]
    }
  }
}
```

### 4.1 路径怎么写 —— 这是最容易错的一步

| 系统 | 写法 | 示例 |
|---|---|---|
| Windows | 正斜杠（**推荐**） | `C:/tools/story-workflow-plugin/server/index.mjs` |
| Windows | 转义反斜杠 | `C:\\tools\\story-workflow-plugin\\server\\index.mjs` |
| macOS / Linux | 正常路径 | `/Users/me/tools/story-workflow-plugin/server/index.mjs` |

**Windows 上不要写单反斜杠** `C:\tools\...` —— JSON 里 `\t` 是制表符，路径会被悄悄改坏，
客户端不报错、只是连不上，极难排查。

**`args` 里必须是绝对路径。** 相对路径会按客户端的工作目录解析，几乎必错。

### 4.2 各客户端把配置放哪

| 客户端 | 位置 |
|---|---|
| Claude Desktop | 设置 → 开发者 → 编辑配置（`claude_desktop_config.json`） |
| Cursor | 项目级 `.cursor/mcp.json`，或全局 `~/.cursor/mcp.json` |
| Cline / Roo Code | 扩展设置 → MCP Servers |
| Zed | `settings.json` → `context_servers`（结构不同，见 INSTALL.md） |

**改完必须让用户完全退出客户端再打开。** Windows 上 Claude Desktop 要在托盘里也退出，
只关窗口不算。

### 4.3 仓库根的三个配置文件，别混用

仓库根有三份配置，**面向三类不同的宿主**。混用是接入失败最常见的原因，
而且失败方式通常是**静默的**——不报错，只是不加载。

| 文件 | 格式 | 给谁用 | 关键点 |
|---|---|---|---|
| `plugin.json` + `mcp.json` | **Agent Plugins 1.0.0** | 支持该标准的插件宿主（ChatGPT 桌面端、Codex、Cursor、Copilot、VS Code、Kiro 等） | 把**目录**交给宿主，别手工搬配置。`mcp.json` 里用 `${PLUGIN_ROOT}` |
| `mcp.client.json` | **纯 MCP**（只有 `command` + `args`） | Claude Desktop、Cline、Roo、Zed、Continue、WorkBuddy 等 | 把 `REPLACE_WITH_ABSOLUTE_PATH` 换成真实绝对路径 |
| 上面第 4 节那段 JSON | 同上，内联版 | 想直接粘、不想找文件时 | 与 `mcp.client.json` 等价 |

**为什么不能把 `mcp.json` 直接给通用客户端**

`mcp.json` 里除了 `command` / `args`，还有 `$schema`、`type: "stdio"`、`cwd`。
这三样是 Agent Plugins 规范要求的，但**通用 MCP 客户端只认 `command` 和 `args`**，
遇到多余字段有两种退化方式：

- 宽松的：忽略多余字段，能用；
- 严格的：整条配置判为非法，**静默跳过，不报错**。你以为配好了，其实服务根本没起。

实测过的例子：某宿主把带 `type` / `cwd` 的那条配置整个忽略，日志里连进程都没 spawn；
换成只有 `command` + `args` 之后立刻连上。所以**给通用客户端就用 `mcp.client.json`**。

**判断方法**

- 客户端说自己「支持 Agent Plugins」→ 直接指向仓库根目录。
- 客户端只说自己「支持 MCP」→ 用 `mcp.client.json`，改成绝对路径。
- 不确定 → 先按 `mcp.client.json` 走，最保守。

`${PLUGIN_ROOT}` 与 `${PLUGIN_DATA}` 由宿主注入，插件自己不能伪造，也不要写进 `env`
（官方 schema 明确禁止这两个键出现在 `env` 里）。**通用客户端不认这两个变量**，
写进去会被当字面量，结果就是找不到文件。

### 4.4 还有一道「信任闸门」，别漏

不少宿主不会自动加载新出现的外部 MCP 服务，需要在它自己的界面里**保存一次 + 点信任**
（有的叫「批准」「允许」）。典型表现：

- 配置文件写了、JSON 也合法，但工具列表里没有；
- 直接改配置文件**不会**触发宿主的配置刷新；
- 在宿主 UI 里保存并确认后，进程才真正被 spawn。

所以：**改完配置文件，如果宿主有「连接器 / MCP」管理界面，进去保存一次并点信任。**
只改文件、不点信任，等于没配。

### 4.5 界面出不来，先查这一条

本插件带一个 MCP Apps 工作台（`ui/dist/index.html`，1.2 MB 单文件）。
它由一个工具触发加载：`platform_open`，声明了 `resourceUri`。

**如果宿主支持 MCP Apps 但界面不显示**，按顺序查：

1. `platform_open` 对模型是否可见（`_meta.ui.visibility` 必须含 `model`）。
   若只写 `['app']`，模型看不见 → 没人调 → 宿主不拉资源 → 界面永不出现。
   资源本身是好的：`resources/list` 能列出、`resources/read` 能读到字节。
2. 宿主是否愿意渲染 MCP App（有的宿主只当普通工具集用，不渲染 UI）。
3. 宿主是否有侧栏/面板需要展开。

**界面不显示不影响功能。** 全部 54 个业务工具都是模型可见的，
可以在对话里直接完成建档、写稿、判据校验，只是没有可视化工作台。

---

## 5. 接通之后，第一句话该说什么

**不要**让用户自己想。给他这两句里的一句：

```
我想做一部电影，帮我立项。
```

或者先看成品：

```
把《三人行》示例导入，我想看看填满的十四段是什么样。
```

第二句会**新建一个独立项目**（不会动用户已有的任何项目），
把 158 份完整交付物复制进去。适合让他先看懂「填满的一套长什么样」。

---

## 6. 每个工具的调用顺序（照这个走，不要自己发明）

### 6.1 任何创作动作之前

```
project_status   → 看现在到哪段、下一段是什么、哪些段本形态不走
stage_spec       → 取这一段的判据和小节结构
```

**这两个不要跳过。** 凭记忆说「现在该写剧本了」经常会错——形态不同，
段结构、文件命名、判据都不一样。

### 6.2 写一份交付物

```
create_deliverable   → 生成骨架（人物/场景/分集要传 label 说明是哪个人/哪场/第几集）
（填内容）
check_deliverable    → 哪些必填小节还空着
judge_deliverable    → 过判据；SKIP 的项如实说「机器判不了」，不要美化
```

### 6.3 改已有文件

```
read_file → 改 → save_doc
```

`save_doc` 是**整篇覆盖**，不是打补丁。三件事必须知道：

1. **空正文会被拒**。要清空请直接删文件。
2. **删掉超过 20% 会要求 `confirmShrink=true`**。这是防止「AI 补一行、结果删掉两千行」。
   真要大改，先跟用户确认，再带这个参数重调。
3. **每次保存都会留一版历史**，返回里带 `versionId`。改坏了用 `list_versions` → `diff_version` → `restore_version` 退回去。

### 6.4 写下一集之前

```
read_continuity(ep = 这一集)      → 读上一集末的四项状态
（写剧本）
append_continuity(ep, time, place, look, prop)  → 只记这一集**变了**的东西
```

**不要复制上一行。** 上一行写「左额有疤」、这一集疤消失了，复制过来就永远发现不了。

---

## 7. 会遇到的失败，以及怎么处置

| 报错 / 现象 | 真正原因 | 怎么办 |
|---|---|---|
| 工具列表里没有 `story-workflow` | 配置 JSON 语法错（多逗号/少引号），客户端会**静默失败** | 用 JSON 校验器过一遍；再检查绝对路径；再确认真的重启了 |
| 能列工具，一调用就报错 | 多数是 `workingDir` 传了不存在的目录 | 先 `list_projects` 确认数据根存在 |
| `模板目录不存在…先跑 buildTemplates` | 用的是旧包 | 重新 clone 最新版（0.5.0 起模板随包走） |
| `专业知识不可用` / `没找到知识库` | **正常**，包内不含方法论文库 | 告诉用户这是设计如此，其余功能不受影响；不要谎称已加载 |
| 保存后内容少了 | 触发过 `confirmShrink`，或覆盖了别的内容 | `list_versions` → `restore_version` 退回 |
| 两个窗口同时改同一文件 | **已知限制**：后写的覆盖先写的，没有冲突提示 | 明确告知用户这是当前版本的边界；靠历史版本救回 |

### 不要做的四件事

1. **不要假设 `tools/` 目录存在。** 安装包里没有源码、没有批处理脚本。
   Skill 里提到的开发期脚本，使用者手上都没有。
2. **不要假设 `mcp-server/` 存在。** 只有打包好的 `server/index.mjs`。
3. **不要自己去改 `server/index.mjs`。** 那是构建产物，改了下次更新就没了。
4. **不要在用户没同意时动他的作品目录。** 导入示例会自动新建项目，其余一切写入都要他明确要求。

---

## 8. 数据在哪

用户在 `create_project` 时指定的目录就是数据根：

```
<数据根>/
├── _projects.json
├── projects/<项目名>/
│   ├── _project.json
│   ├── _state/            锁定设定、决策、连续性、版本历史
│   └── 01-立项/ … 14-终检/
├── _trash/                删掉的项目（可恢复）
└── templates/             首次铺模板时生成
```

- **全在本地**，不联网、不上传、不要账号。
- 每个项目一个目录，操作 A 不会碰 B。
- 删项目是**软删除**（进 `_trash/`），能 `restore_trash` 恢复。
- 备份 = 复制整个数据根。

---

## 9. 一句话总结给用户

如果他只想听一句：

> 装 Node 20+，clone 下来，把 `node <路径>/server/index.mjs` 写进客户端的 MCP 配置，重启。不用 npm install。

---

## 10. 你自己要守的线

- **说清楚能力边界。** 网页版用不了就是网页版用不了，不要含糊。
- **不要承诺没验证过的事。** 「应该可以」在安装问题上等于没说。
- **报错原文优先。** 用户贴了报错，先看原文再判断，不要凭症状猜。
- **改用户作品前先问。** 这是他的稿子，不是测试数据。
