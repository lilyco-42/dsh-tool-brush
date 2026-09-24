# dsh-tool-brush

**Model-facing [brush](https://brush.sh) shell tool for DeepSeek Harness — real bash syntax on Windows, no Nushell translation layer.**

Registers a `brush` tool for every agent in the profile. Each call runs a fresh
[brush](https://brush.sh) (Bo[u]rn[e] RUsty SHell, a Rust-native bash-compatible shell)
process with `-c`, so bash scripts behave like bash — not like an alias simulator.

## Why

- DSH's native `bash` tool is disabled on Windows (needs WSL).
- `pwsh` is slow and its syntax differs from bash.
- `dsh-tool-nu` emulates bash with Nushell aliases, which breaks on real bash syntax
  (`x=1`, `$(cmd)`, `2>&1`, `for ... do ... done`, `&&` / `||`, ...).
- brush is a **real bash implementation**: variables, command substitution, pipelines,
  redirection, control flow, functions, arrays, arithmetic — all native.

## Features

- Full bash syntax: `x=1; echo $x`, `$(pwd)`, `` `pwd` ``, `2>/dev/null`, `2>&1`,
  `&&` / `||`, `if/for/while/case`, `f() {...}`, arrays, `$(( ))`, `$?`, `printf`
- Git for Windows coreutils on PATH when available:
  `ls`, `cat`, `grep`, `wc`, `tr`, `head`, `tail`, `tee`, `xargs`, `find`, `sed`, `awk`, ...
- Fresh shell per call (`--no-config`): deterministic, no state leakage
- Background jobs, sandbox escalation, spill, timeout — same contract as other shell tools

## Install

```sh
# Recommended — works out of the box
dsh plugin --profile web add github:lilyco-42/dsh-tool-brush
```

Restart `dsh web`. The `brush` tool appears in every agent's catalog.

### Installing from a local checkout

`dsh plugin add <path>` installs through pnpm's `link:` protocol, so the profile points at your checkout instead of copying it. Node then resolves this plugin's `@deepseek-ai/*` imports from the **checkout's** directory tree, which never reaches the profile's `node_modules` — and the plugin fails to load with:

```
Cannot find package '@deepseek-ai/schemastery' imported from <checkout>/lib/index.js
```

Make dsh's own modules reachable from the checkout (Windows, no admin required):

```powershell
New-Item -ItemType Junction `
  -Path "<checkout>\node_modules\@deepseek-ai" `
  -Target "$env:USERPROFILE\.dsh\profiles\node_modules\@deepseek-ai"

dsh plugin --profile web add <checkout>
```

`<DSH_HOME>\profiles\node_modules\@deepseek-ai` is the directory dsh provisions itself (one link per in-box package, ~240 of them), so the plugin resolves the **same module instances** dsh is running.

> Do not "fix" this with a plain `pnpm install` inside the checkout. It pulls a second copy of `@deepseek-ai/dsh-tools`, `dsh-llm`, … into the checkout. The plugin then loads, but `instanceof HarnessError` and `Symbol()`-keyed lookups compare against a different module instance than the host's.

## Requirements

- [brush](https://brush.sh) 0.4+ on PATH or at the candidate paths in `lib/index.js`
  (e.g. `scoop install brush`, `cargo install brush`)
- Optional but recommended: Git for Windows (for GNU coreutils)
- DeepSeek Harness 0.1.0-rc.6+
