# 安装与排障

## 一、先确认 Node

```bash
node -v
```

要 **v20.6.0 或更高**。低了就升。没有 Node 就去 <https://nodejs.org> 装 LTS。

服务端是打好的单文件，**不需要 npm install，不需要编译**。

---

## 二、放到一个固定位置

```bash
git clone https://github.com/toinhaandrade637-wq/story-workflow-plugin.git
```

或者下载 ZIP 解压。

建议放在一个不会被随手清掉的地方，比如：

- Windows：`C:\tools\story-workflow-plugin`
- macOS / Linux：`~/tools/story-workflow-plugin`

**别放在「下载」或临时目录**——清空下载文件夹 = 插件消失。

先验证服务端能起来（这一步不接客户端，纯自检）：

```bash
node /你的路径/server/index.mjs
```

正常表现是**卡住不动、没有任何输出**——它在等 stdin 的 MCP 消息。这就对了，`Ctrl+C` 退出。

如果报 `Cannot find module`，说明路径不对；如果报语法错误，说明 Node 版本太低。

---

## 三、接进客户端

### Claude Desktop

1. 打开设置 → 开发者 → 编辑配置
2. 在 `mcpServers` 里加：

```json
{
  "mcpServers": {
    "story-workflow": {
      "command": "node",
      "args": ["C:/tools/story-workflow-plugin/server/index.mjs"]
    }
  }
}
```

3. 完全退出 Claude Desktop（Windows 要在托盘里也退出），重新打开
4. 对话框里出现工具图标就通了

Windows 路径二选一：正斜杠 `C:/tools/...`（推荐），或转义反斜杠 `C:\\tools\\...`。

### Cursor

项目级：在项目根建 `.cursor/mcp.json`；全局：`~/.cursor/mcp.json`。内容同上。

### Cline / Roo Code

扩展设置 → MCP Servers → 添加，把 `command` 填 `node`，`args` 填服务端绝对路径。

### Zed

`settings.json` 里：

```json
{
  "context_servers": {
    "story-workflow": {
      "command": { "path": "node", "args": ["C:/tools/story-workflow-plugin/server/index.mjs"] }
    }
  }
}
```

### 其他客户端

只要是**支持 MCP stdio 的本地客户端**，配置形状基本一致：`command` = `node`，`args` = `[服务端绝对路径]`。

包里的 `mcp.json` 用的是 `${PLUGIN_ROOT}` 占位符。如果你的客户端认识这个变量（部分插件式宿主会替换），可以直接用；不认识就**换成绝对路径**，否则会报找不到文件。

---

## 四、第一次用

跟它说：

```
我想做一部电影，帮我立项。
```

它会问你一句话故事，然后：

1. `create_project` 建目录
2. `apply_template` 铺该形态的空骨架
3. 一段一段往下推

想先看成品长什么样：

```
把《三人行》示例导入，我想看看填满的十四段是什么样。
```

会复制成独立项目，不动任何已有内容。

---

## 五、数据在哪，怎么备份

你在第一步建项目时指定的目录就是数据根：

```
你的目录/
├── _projects.json          上次打开的是哪个
├── projects/<项目名>/       每个项目一个目录
├── _trash/                 删掉的项目（可恢复）
└── templates/              铺模板用
```

**备份就是复制整个目录。** 没有数据库、没有云同步、没有隐藏状态。

版本历史在 `projects/<项目名>/_state/versions/`，最多 40 版。

---

## 六、卸载

1. 从客户端配置里删掉 `story-workflow` 这一段
2. 删掉 clone 下来的目录

**你的作品在数据目录里，卸载插件不会删它。** 要一起删就手动删数据目录。

---

## 七、常见问题

**工具列表里没有 story-workflow**

- 配置 JSON 有语法错误（多一个逗号、少一个引号）——客户端会静默失败，用在线 JSON 校验器过一遍
- 路径不对：`args` 里必须是**绝对路径**，指向 `server/index.mjs`
- 没重启客户端
- Node 不在 PATH 里：把 `command` 改成 Node 的绝对路径

**能列工具，但一调用就报错**

- 看是不是 `workingDir` 传了不存在的目录。绝大多数工具都要一个已存在的工作目录
- `apply_template` 报「模板目录不存在」：说明包不完整，重新 clone

**"专业知识不可用"**

这是正常的。包内不含方法论文库，`stage_knowledge` 会明确告诉你缺哪几页，其余功能不受影响。

**保存后内容不对 / 少了**

每次保存都进历史版本。用 `list_versions` 看，`diff_version` 比，`restore_version` 退。

**两个窗口同时改同一份文件**

当前版本**没有并发冲突保护**，后保存的会覆盖先保存的。历史版本里两份都在，能救回来，但不会提前拦你。多人协作请勿同时开。

---

## 八、报问题

带上这些信息：

- 操作系统 + `node -v`
- 哪个客户端
- 哪一步失败，报错原文
- 能不能列出工具（`listTools` 有没有返回）
