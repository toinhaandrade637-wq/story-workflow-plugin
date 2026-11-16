# 影视全流程 · Story Workflow

把一个创作念头，一路推到能拍的剧本、能落地的分镜、能递出去的项目书。

它不是"一键生成小说"的黑盒。它是一条**有工序的产线**：十四段，每段只交一份东西，每份都带判据——写完能告诉你哪里还空着、哪里自相矛盾。你（或你的 AI）负责写，它负责不让你漏、不让你前后打架。

> **先看你用的是什么客户端。**
>
> | 你的客户端 | 怎么装 |
> |---|---|
> | 支持 **Agent Plugins 1.0.0** 的（ChatGPT 桌面端 / Codex / Cursor / Copilot / VS Code / Kiro） | 把仓库目录整个交给它，**不用手写配置**。根目录的 `plugin.json` + `mcp.json` 是标准格式，已通过官方 schema |
> | 只支持 **MCP** 的（Claude Desktop / Cline / Roo / Zed / Continue / WorkRally） | 用[下方手工配置](#30-秒接入)，**把路径换成你的绝对路径** |
> | **纯网页聊天**（ChatGPT 网页版、Claude.ai 网页版） | [接不了](#关于-chatgpt)，原因写在那节 |

> **如果你是 AI 助手，正在帮用户装这个** → 直接读 [AGENTS.md](AGENTS.md)。
> 那一页讲的是：能不能用、Node 怎么查、路径怎么写不踩坑、两种清单别混用、报错怎么处置、工具按什么顺序调。

---

## 它能干什么

**十四段工序**

| # | 段 | 交什么 |
|---|---|---|
| 1 | 立项 | 一句话故事、类型、体量、四步主干 |
| 2 | 总论 | 导演阐述八维、结构骨架、账本 |
| 3 | 生活场 | 四维物理生活场、五感、铁律 |
| 4 | 人物 | 一人一档，A/B/C/D 分级 |
| 5 | 场景 | 一景一档，拓扑与陈设状态 |
| 6 | 道具 | 编码、流转链、出现场次 |
| 7 | 分集 | 全局大纲 + 单集场景纲 |
| 8 | 细节 | 伏笔与闲笔，首末场与回收 |
| 9 | 剧本 | Fountain 正文 |
| 10 | 分镜 | 逐镜七列（景别/焦段/运动/内容/声音/时长/备注） |
| 11 | 阐述 | 重场戏的视听落地 |
| 12 | 项目书 | 可对外的一页纸 |
| 13 | 宣发 | 定调句、三版简介、预告结构、短视频 |
| 14 | 终检 | 连续性对账、复盘 |

**四形态，各走各的**

| 形态 | 单位 | 特有约束 |
|---|---|---|
| 剧集 `drama` | 集 | 集末钩子、季弧光、平台分账口径 |
| 短剧 `short` | 集（1–3 分钟） | 前 25 字钩子、付费卡点、投流素材 |
| 电影 `film` | 场 / 幕 | 不用"集"字，八维导演阐述，四步体检 |
| 小说 `novel` | 卷 / 章 | 不要求镜头与分镜，文风七项 |

判据按形态分化：电影不会被要求填"第几集"，小说不会被要求做分镜，短剧不会套长剧的季弧光。

**54 个工具**：建项目、铺模板、写文件、查缺、判据校验、版本回退、软删除与恢复、连续性台账、项目书渲染、导出 Word、打包设定集。

---

## 30 秒接入

前提：**Node.js 20.6 或更高**（`node -v` 看一眼）。服务端是打包好的单文件，**不需要 npm install**。

### 1. 拿到它

```bash
git clone https://github.com/toinhaandrade637-wq/story-workflow-plugin.git
```

放在哪都行，记住这个绝对路径。

### 2. 接进你的 AI 客户端

**分两种情况，先确认你属于哪一种。**

#### A. 你的客户端支持 Agent Plugins（ChatGPT 桌面端 / Codex / Cursor / Copilot / VS Code / Kiro）

不用手写配置。**把 clone 下来的整个目录交给它**——它会读根目录的 `plugin.json`，
按 `mcp.json` 起 `server/index.mjs`。路径由宿主用 `${PLUGIN_ROOT}` 自己拼。

唯一要确认的是 Node 在宿主的 PATH 里。装完让它列一下工具，看到 `story-workflow` 就通了。

#### B. 你的客户端只支持 MCP（Claude Desktop / Cline / Roo / Zed / Continue）

把下面这段加进它的配置，**把路径换成你 clone 下来的真实绝对路径**：

```json
{
  "mcpServers": {
    "story-workflow": {
      "command": "node",
      "args": ["/你clone到的路径/story-workflow-plugin/server/index.mjs"]
    }
  }
}
```

Windows 的路径要写成 `"C:/tools/story-workflow-plugin/server/index.mjs"` 或 `"C:\\\\tools\\\\...\\\\server\\\\index.mjs"`。

> 这类客户端**不认** `${PLUGIN_ROOT}`，直接抄根目录 `mcp.json` 会找不到文件。必须换成绝对路径。

**各客户端配置在哪**

| 客户端 | 配置文件 |
|---|---|
| Claude Desktop | `claude_desktop_config.json`（设置里「开发者」→ 编辑配置） |
| Cursor | `.cursor/mcp.json`（项目级）或全局 `~/.cursor/mcp.json` |
| Cline / Roo | 扩展设置里的 MCP Servers |
| Zed | `settings.json` 的 `context_servers` |
| WorkRally | 插件目录，或直接用这份 `mcp.json` |

改完**重启客户端**。让它列一下工具，能看到 `story-workflow` 就通了。

### 3. 说第一句话

```
我想做一部电影，帮我立项。
```

它会先问你一句话故事，然后建项目、铺骨架，一段一段往下推。

---

## 关于 ChatGPT

说清楚一件事，免得你白折腾。**「ChatGPT」是两回事，别混：**

**ChatGPT 网页版（聊天框）—— 跑不了。**
它只连远程 MCP（HTTPS），不接受「在你电脑上起一个本地进程」这种 stdio 方式。
想让网页版用，需要有人把这个服务部署成带 HTTPS 的远程端点，并解决身份与存储隔离——
**本项目没有提供这部分**，别按本地配置去连。

**ChatGPT 桌面端 / Codex —— 有机会用。**
它们支持 **Agent Plugins 1.0.0**，而本仓库根目录就是标准包：

```
plugin.json   ← 通过 agent-plugins.org/schemas/1.0.0/plugin.schema.json
mcp.json      ← 通过 agent-plugins.org/schemas/1.0.0/mcp.schema.json，声明 type: "stdio"
skills/       ← 四个 Agent Skill，标准 skills/<name>/SKILL.md 结构
```

把整个仓库目录交给支持该标准的宿主，它会自己读 `plugin.json`、起 `server/index.mjs`。
不用手写配置，也不用改路径（`${PLUGIN_ROOT}` 由宿主注入）。

同样的话适用于任何云端产品：**能不能用，取决于宿主肯不肯在本机起 `node` 进程，
不取决于这个包本身。**

---

## 两种清单，别混用

仓库根同时有两个清单，面向两类宿主。搞混是接入失败最常见的原因：

| 文件 | 格式 | 给谁 | 路径怎么写 |
|---|---|---|---|
| `plugin.json` + `mcp.json` | Agent Plugins 1.0.0 | 支持该标准的插件宿主 | `${PLUGIN_ROOT}`，宿主会替换 |
| 本文档里的手工 JSON 片段 | 通用 MCP 配置 | Claude Desktop / Cline / Roo / Zed / Continue | **必须绝对路径**，这类客户端不认 `${PLUGIN_ROOT}` |

`plugin.json` 的 schema 是**封闭的**（`additionalProperties: false`），只允许
`$schema` / `name` / `version` / `description` / `author` / `homepage` / `repository` /
`license` / `keywords` / `extensions`。多写一个字段整份清单就校验失败，宿主直接不加载。

---

## 数据放在哪

你在客户端里让它建项目时，会指定一个工作目录。所有东西都写在那：

```
你的目录/
├── _projects.json
├── projects/
│   └── 我的电影/
│       ├── _project.json
│       ├── _state/          ← 锁定设定、决策、连续性、版本历史
│       ├── 01-立项/
│       ├── …
│       └── 14-终检/
├── demo/                    ← 首次导入示例时会复制
└── templates/               ← 首次铺模板时会用到（包内自带一份）
```

- **全在本地**，不联网、不上传、不要账号。
- 每个项目一个目录，操作 A 项目不会碰 B 项目一个字节。
- 删项目是**软删除**，进 `_trash/`，能恢复。
- 每次保存都留一版历史，最多 40 版，能 diff、能回退。

---

## 包里有什么

```
story-workflow-plugin/
├── server/index.mjs        零依赖服务端（2.0 MB，已打包）
├── skills/                 四个 Skill：分诊 / 点子 / 大纲 / 剧本
├── data/demo/              唯一示例《三人行》（158 份，完整成品）
├── data/templates/         四形态空骨架（58 份）
├── ui/dist/index.html      MCP Apps 工作台（宿主支持时显示）
├── AGENTS.md               给 AI 助手的接入手册（配置、排错、调用顺序）
├── llms.txt                机器可读的仓库摘要
├── mcp.json                客户端配置模板
├── plugin.json             插件清单
├── RELEASE_MANIFEST.json   每个文件的 sha256
├── LICENSE                 MIT
├── INSTALL.md              安装、排障、卸载
├── PRIVACY.md              数据去哪了
└── CHANGELOG.md
```

**这份包里没有源码，也没有开发脚本。** 只有运行时需要的东西。
Skill 里偶尔提到 `tools/*.ts` 这类开发期批处理脚本——那是开发仓库里的工具，
使用者手上没有，不要假设它存在。

**示例只有《三人行》一部**。它是用来让你看懂"填满的一套长什么样"的，不是模板——不要在上面直接改，导入时会复制成独立项目。

---

## 已知边界

- **并发编辑没有冲突保护**：两个窗口同时改同一份文件，后保存的会覆盖先保存的。每次保存的历史版本还在，能回退，但不会弹"冲突了"。多人协作先别用。
- **专业知识库要自己提供**：包内不含方法论文库。没有它时，`stage_knowledge` 会明确告诉你"不可用"，其余功能照常。这是设计，不是故障。
- 需要 Node 20.6+；更低版本起不来。

---

## 许可

MIT。见 [LICENSE](LICENSE)。

示例作品《三人行》为原创虚构内容，随包提供用于演示工作流。
