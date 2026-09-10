# dsh-prompt-pack

一个可以被 [dsh-prompt-manager](https://github.com/lolkda/dsh-ctf-prompt) 订阅的 DeepSeek Harness（DSH）提示词包。仓库里放的是 **system prompt section** 的 markdown 正文，订阅方检查更新、按文件挑选、再应用到自己机器上。

当前包含两条：

| 文件 | 标题 | 默认顺序 | 内容 |
|---|---|---|---|
| `contract.md` | CTF 契约 | 10 | CTF / 竞赛沙箱作业契约：核心约定、证据优先级、工作流、工具与结果呈现、协作与网络语境 |
| `fastctx.md` | FastCtx 工具路由 | 20 | 优先使用 `mcp__fastctx__*`（inspect_local_file / grep / glob / replace / run）而不是 shell 等价物的工具路由指引 |

## 订阅方式

在 DSH 的 **设置 → 提示词 → 来源** 里添加一个来源：

- 仓库：`lolkda/dsh-prompt-pack`
- ref：`main`（分支 / tag / commit 都行）
- 镜像：留空表示直连，也可以填自己的 `https://` 前缀镜像

添加后点 **检查更新**，看到每个文件的 `+n / −m` 变更行，勾选要的文件再点 **应用**。

或者直接写进 `$DSH_HOME/settings.yaml` 的 `prompt-manager:` 段：

```yaml
prompt-manager:
  sources:
    - id: lolkda-dsh-prompt-pack      # 省略也可以，插件会从 owner/repo 派生
      repo: lolkda/dsh-prompt-pack
      ref: main
      mirror: ''
      enabled: true
```

**新导入的条目默认关闭**，打开开关后才会注入 system prompt —— 远端正文不会在你没确认之前进入对话。

## 仓库要满足的格式

插件订阅的仓库根目录必须有一份 `prompt-manager.json` 清单，因为列举远端目录要么走 GitHub Contents API（有配额），要么解析整仓 tarball，两种都比「作者多写一个文件」贵：

```json
{
  "prompts": [
    { "file": "contract.md", "title": "CTF 契约", "order": 10 },
    { "file": "fastctx.md", "title": "FastCtx 工具路由", "order": 20 }
  ]
}
```

- `file`：仓库内相对路径，必须以 `.md` 结尾，不能以 `/` 开头、不能含 `..`
- `id`：可省略，缺省取文件名（订阅方会再加 `<来源 slug>-` 前缀，避免和本地条目撞名）
- `title` / `order` / `enabled`：都可省略；`order` 缺省时由订阅方在列表末尾分配
- 整个清单是远端内容，每个字段都会被校验：非法路径、`..`、非 `.md`、重复文件、超过 50 条都会拒绝

## 更新

改任一 markdown 后提交到 `main` 即可。订阅方点「检查更新」时先用 `github.com/<repo>/commits/<ref>.atom` 取 head sha（零 API 配额），sha 没变就直接说「已是最新」，变了才逐文件发条件请求（`If-None-Match`）。应用前的旧正文会留在 `previous/`，一次「还原」就能退回去。

## 来源

这两份正文最初是 `dsh-prompt-manager` 插件自带的种子提示词，现在独立成一个可订阅的包，方便在多台机器、多个 profile 之间同步同一份契约。

## License

MIT
