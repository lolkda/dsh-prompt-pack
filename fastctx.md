# MCP Tool Routing

## Local file inspection

For reading, searching, and finding local files, prefer the FastCtx MCP
server's own tools — `mcp__fastctx__inspect_local_file`,
`mcp__fastctx__grep`, and `mcp__fastctx__glob` — over shell
equivalents such as `cat`/`Get-Content`, `rg`/`findstr`/`Select-String`,
and `dir`/`ls -R`.
Use FastCtx file tools directly for local-file operations, including when a
local reference is URI-shaped; pass the equivalent plain absolute filesystem path.
Read only what the task needs. When you need several files, pass them to
one `mcp__fastctx__inspect_local_file` call as files=[{"path": ...}, ...] instead
of one call per file. The last line of every result says `Complete` or
`Partial` — continue only with the exact parameters a `Partial` note
provides.

### Batch replacement

Use `mcp__fastctx__replace` for mechanical find-and-replace across files.
It preserves each file's encoding and line endings, supports dry-run previews,
and rejects concurrent changes before writing. Use the harness's own `edit` for
targeted changes to existing files, and `write` for generated or fully
rewritten content.

### Shell commands

Prefer `mcp__fastctx__run` over the built-in shell for terminal work: it
executes with bash (Git Bash on Windows), so always write POSIX bash —
never PowerShell syntax. Use the harness's `pwsh` tool when you specifically
need PowerShell.

`mcp__fastctx__run` has no working-directory parameter and inherits the harness
host's cwd rather than the task workspace, so pass absolute paths instead of
relying on relative ones.

Commands must be non-interactive (no TTY): use flags like -y
or --no-edit, and expect editors/pagers to be disabled. For anything
that may outlast `mcp__fastctx__run`'s four-minute maximum, use
`mcp__fastctx__run_background`, check on it with
`mcp__fastctx__job_output`, and stop it with `mcp__fastctx__job_kill`.
Background jobs run independently of this session and survive restarts;
rediscover an earlier job with `mcp__fastctx__job_list` and read its output by
job_id. A non-zero exit code is a normal result. The last line of every result
says `Complete` or `Partial`.
