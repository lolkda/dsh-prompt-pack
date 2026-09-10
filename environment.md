# Machine environment

Detected on the machine running the DSH host process, and refreshed whenever the prompt plugin mounts. Every `{{...}}` below is a measured value rather than a default impression — when one of them conflicts with what you observe, trust the observation.

## System

- {{os}}, release `{{os_release}}` (`{{platform}}` / `{{arch}}`)

## Shells

- **bash** `{{bash}}` — write POSIX syntax: `&&`, `$VAR`, forward slashes.
- **PowerShell** `pwsh` `{{pwsh}}`
  - Use PowerShell 7 or newer. Windows PowerShell 5.1 is not UTF-8 by default — file writes go through the system ANSI code page, and what it pipes to native programs differs as well — so non-ASCII text is mangled or fails outright. Do not fall back to it.
  - A value of `(not installed)` above means this machine has no usable PowerShell; that is not a reason to reach for 5.1.

## Version control

- **git** `{{git}}`
- A repository with no committer identity fails its first commit with `unable to auto-detect email`. Check before committing (`git config user.email`), and where the repository already has commits, reuse their author rather than inventing one.

## Languages

- **Node** `{{node}}`
- **Python** `{{python}}` — the command is normally `python`. On Windows `python3` may be an App Execution Alias stub that starts, prints nothing, and reports no error, so a silent `python3 --version` is not evidence that Python is missing.
