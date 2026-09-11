# dsh-prompt-pack

一个可以被 [dsh-prompt-manager](https://github.com/lolkda/dsh-prompt-manager) 订阅的 DeepSeek Harness（DSH）提示词包。仓库里放的是 **system prompt section** 的 markdown 正文，订阅方检查更新、按文件挑选、再应用到自己机器上。

当前包含三条：

| 文件 | 标题 | 默认顺序 | 内容 |
|---|---|---|---|
| `fastctx.md` | FastCtx 工具路由 | 20 | 优先使用 `mcp__fastctx__*`（inspect_local_file / grep / glob / replace / run）而不是 shell 等价物的工具路由指引 |
| `ctf.md` | CTF 沙箱契约 | 30 | CTF / 竞赛沙箱作业契约：核心约定、证据优先级、工作流、工具、协作与网络语境 |
| `engineering.md` | 工程判断 | 40 | 破坏性改动什么时候该作为推荐方案，而不是默认发兼容补丁 |

机器环境那条**不在这里**：它作为 `dsh-prompt-manager` 的内置条目随插件发布（正文与变量由插件自带，开箱即用），不再需要订阅。这个包只放按部署定制的正文。

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
    { "file": "fastctx.md", "title": "FastCtx 工具路由", "order": 20 },
    { "file": "ctf.md", "title": "CTF 沙箱契约", "order": 30 },
    { "file": "engineering.md", "title": "工程判断", "order": 40 }
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

这三份正文最初都来自 `dsh-prompt-manager` 插件的种子提示词，现在独立成一个可订阅的包，方便在多台机器、多个 profile 之间同步同一份契约。机器环境那条已改由插件内置发布（值在挂载时探测），见插件的 README。

原先仓库里只有一份 `contract.md`，里面拼着三件不相干的东西（种子的历史遗留）：通用输出规范、CTF 沙箱契约、工程判断。现已按用途拆开：CTF 的七节移进 `ctf.md`，`Engineering Judgment` 独立成 `engineering.md`，`contract.md` 随之删除 —— 其中排版规则和最终答复风格那两节不再保留（要找回看 git 历史里的 `contract.md`），与 CTF 无关的 `Presenting Results` 一节也一并删去。

顺序号保留原值（`fastctx` 20 / `ctf` 30 / `engineering` 40），空出来的 10 是删掉的 `contract.md`：这样订阅方那边其余条目的相对顺序不变。订阅方「检查更新」时会看到 `contract.md` 条目消失、`ctf.md` 与 `engineering.md` 出现，属预期。

## License

MIT
