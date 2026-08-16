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
dsh plugin --profile web add /path/to/dsh-tool-brush
```

Restart `dsh web`. The `brush` tool appears in every agent's catalog.

## Requirements

- [brush](https://brush.sh) 0.4+ on PATH or at the candidate paths in `lib/index.js`
  (e.g. `scoop install brush`, `cargo install brush`)
- Optional but recommended: Git for Windows (for GNU coreutils)
- DeepSeek Harness 0.1.0-rc.6+
