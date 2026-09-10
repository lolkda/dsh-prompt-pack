# dsh-prompt-pack

一个可以被 [dsh-prompt-manager](https://github.com/lolkda/dsh-prompt-manager) 订阅的 DeepSeek Harness（DSH）提示词包。仓库里放的是 **system prompt section** 的 markdown 正文，订阅方检查更新、按文件挑选、再应用到自己机器上。

当前包含三条：

| 文件 | 标题 | 默认顺序 | 内容 |
|---|---|---|---|
| `environment.md` | 机器环境 | 5 | 本机系统与 shell / 工具链版本，值全部由 `{{…}}` 变量填充（需要订阅方先配好探测，见下） |
| `contract.md` | CTF 契约 | 10 | CTF / 竞赛沙箱作业契约：核心约定、证据优先级、工作流、工具与结果呈现、协作与网络语境 |
| `fastctx.md` | FastCtx 工具路由 | 20 | 优先使用 `mcp__fastctx__*`（inspect_local_file / grep / glob / replace / run）而不是 shell 等价物的工具路由指引 |

## 机器环境这条要先配探测

`environment.md` 是**三端通用**的：它不写死任何一台机器的事实，只引用变量，值由**订阅方自己的机器**在插件挂载时探测填充（`{{os}}` / `{{os_release}}` / `{{platform}}` / `{{arch}}` + 探测出来的 `{{bash}}` / `{{pwsh}}` / `{{git}}` / `{{node}}` / `{{python}}`）。所以订阅后请在 profile 的 `cordis.patch.yml` 里补上这一段：

```yaml
      config:
        environment: true
        probes:
          pwsh:   { command: pwsh, args: ['-NoProfile', '-Command', '$PSVersionTable.PSVersion.ToString()'] }
          bash:   { command: bash, args: ['--version'], pattern: 'version ([0-9.]+)' }
          git:    { command: git,  args: ['--version'], pattern: '([0-9]+\.[0-9]+\.[0-9]+)' }
          node:   { command: node, args: ['--version'], pattern: 'v?([0-9.]+)' }
          python: { command: python, args: ['--version'], pattern: '([0-9.]+)' }
```

两个平台细节：不在 `PATH` 上的工具要写绝对路径（Windows 上宿主进程的 `PATH` 往往只有 `Git\cmd`，没有 `bash.exe`）；`npm` / `pnpm` 这类 `.cmd` 垫片要加 `shell: true`。

**不配会怎样**：变量没注册时，正文里那个 `{{名字}}` 会让 `assemble()` 直接抛错（整个 system prompt 组不出来），而不是渲染成空串。所以要么把探测配齐，要么把这条条目的开关关掉。

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
    { "file": "environment.md", "title": "机器环境", "order": 5 },
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

这三份正文最初是 `dsh-prompt-manager` 插件自带的种子提示词（机器环境那条后来改成了变量驱动的通用版），现在独立成一个可订阅的包，方便在多台机器、多个 profile 之间同步同一份契约。

## License

MIT
