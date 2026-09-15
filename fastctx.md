# MCP Tool Routing

FastCtx is an MCP server with its OWN session cwd (measured: `/d/Personal/Desktop`),
NOT the DSH task workspace. Its `path`/`cwd` arguments default to that directory, so
always pass an absolute `path` (glob/grep) or `cwd` (run) explicitly.

## Local file inspection

For reading, searching, and finding local files, prefer the FastCtx MCP server's own
tools — `mcp__fastctx__inspect_local_file`, `mcp__fastctx__grep`, and
`mcp__fastctx__glob` — over shell equivalents such as `cat`/`Get-Content`,
`rg`/`findstr`/`Select-String`, and `dir`/`ls -R`.

Use FastCtx file tools directly for local-file operations, including when a local
reference is URI-shaped; pass the equivalent plain absolute filesystem path.

- `glob`: `*` matches ONE level only — to descend the tree write `**/`, e.g. `**/*.py`.
  `filter_mode` defaults to `ignore` (plain `.ignore` files only); `all` disables it.
- `grep`/`glob` silently skip binaries and files they cannot decode or lock, and list
  them at the end as `— undecodable`, `— locked by another process`, or
  `— changed while being searched`. A skipped file is NOT an absent file. For those,
  fall back to `inspect_local_file` with `view="hex"`, or a `run` script. IDA-held
  `.id0`/`.id1`/`.nam` are always in this class.
- Read only what the task needs. A `Partial` note supplies the exact parameters for
  the next call — follow them verbatim instead of re-deriving your own.

`inspect_local_file` also accepts `files=[{"path": ..., "offset": ..., "limit": ...}, ...]`
for several files in one call, but the batch shares ONE token budget: give every entry
its own `limit`, and never batch files with very long lines (base64 blobs, compressed
bodies, minified JS) — one long line can exhaust the budget before the second entry is
reached. For those, read one range per call.

Binary, image, and PDF reads are single-file only: `view="hex"` for raw bytes, plus
`pdf_mode`/`pages` for PDFs.

## Parameter mutual exclusion

Both inspection tools have mutually exclusive argument groups, and the checks fire one
at a time — one wrong call fails repeatedly instead of once. Fix every offending
argument in a single retry.

`grep`
- `encoding` is SINGLE-FILE only. For a directory `path`, use `fallback_encoding`.
- On a directory, `fallback_encoding` applies only to files auto-detection cannot
  resolve, and decoding under it is strict: a GBK-log directory given
  `fallback_encoding="utf-8"` silently drops those files. Use `"gbk"` for CJK sources,
  or omit both parameters.
- Do not pass `type` and `glob` for the same extension filter; pick one.

`inspect_local_file`
- `files` is mutually exclusive with `file_path`, and with the TOP-LEVEL `offset`,
  `encoding`, `pages`, `pdf_mode`, and `view`.
- A top-level `limit` IS allowed with `files` and serves as the default for entries
  that omit their own.
- Batch: `files=[{"path": ..., "offset": 1, "limit": 200}, ...]` plus optional
  top-level `limit` — nothing else.
- Single file: `file_path` with `offset`/`limit`, plus `encoding` only when
  auto-detection is not confident.

## Failure recovery

Every FastCtx error names the offending argument and is self-contained. When one says a
parameter "cannot be combined with files" or "only applies to single-file targets",
remove ALL single-file-only arguments in one retry — never peel them off one failure at
a time, and never re-issue an identical call to see whether it works now.

`Partial: 0 of N entries processed` is NOT a read failure: content above it was
returned. Judge by whether content came back and by the continuation parameters the
Partial note supplies.

## Batch replacement

Use `mcp__fastctx__replace` for mechanical find-and-replace across files. It preserves
each file's encoding and line endings, rejects concurrent changes before writing, and
supports `dry_run` — preview first, and set `max_replacements` as a blast-radius guard.
`literal: true` for plain text; add `(?m)` when you need per-line `^`/`$` anchors
(without it they anchor the whole file). Use the harness's own `edit` for targeted
changes to existing files, and `write` for generated or fully rewritten content.

## Shell commands

Prefer `mcp__fastctx__run` over the built-in shell for terminal work: it executes with
bash (Git Bash on Windows), so always write POSIX bash — never PowerShell syntax. Use
the harness's `pwsh` tool when you specifically need PowerShell.

`mcp__fastctx__run` DOES take `cwd` (absolute path). Its default is FastCtx's own
session cwd, not the task workspace, so pass `cwd` — or absolute paths inside the
command — instead of relying on relative ones. It also takes `timeout_ms` (default
120000, hard maximum 240000). Expect `U+FFFD` when an external program emits CJK in the
system code page; re-run with `encoding="gbk"` rather than guessing at the text.

Commands must be non-interactive (no TTY): use flags like -y or --no-edit, and expect
editors/pagers to be disabled. For anything that may outlast `mcp__fastctx__run`'s
four-minute maximum, use `mcp__fastctx__run_background`, check on it with
`mcp__fastctx__job_output`, and stop it with `mcp__fastctx__job_kill`. Background jobs
run independently of this session and survive restarts; rediscover an earlier job with
`mcp__fastctx__job_list` (`status:"all"` to include finished ones) and read its output
by job_id. A non-zero exit code is a normal result. The last line of every result says
`Complete` or `Partial`.
