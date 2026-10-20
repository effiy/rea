---
title: Analysis Efficiency Measurements
tags: [rea, analysis, efficiency, experiments, optimization, benchmarks]
category: experiments
created: "2024-01-01"
updated: "2026-10-20"
source: external
type: index
status: stable
lifecycle: active
---

# 💡 概述

Tracks the experiments for [#2016](https://github.com/morluto/rea/issues/2016).
The production CLI/MCP contracts and defaults remain the compatibility baseline.
Downloaded artifacts, runtime copies, exported Evidence, transcripts and detailed
JSON measurements belong outside the checkout. This document keeps the measured
findings, verification scope and revision references; the scripts and `corpus.json`
keep the reproduction inputs. Generated `results/` is ignored.

### Follow-up documents

- [native PE/DOS entry views and real model verification](native-entry-views.md)
- [EVM descriptor and real-template controls](evm-descriptor.md)
- [EVM interface summary delivery](evm-interface-summary.md)
- [task-scoped preset lifecycle verification](preset-lifecycle.md)
- [EVM descriptor lifecycle acceptance](evm-lifecycle.md)
- [managed artifact delivery and retained views](managed-artifact-summary.md)
- [model tool context and per-response usage](model-context.md)
- [Android package delivery and retained manifest](android-package-summary.md)
- [resource counters at payload completion](resource-counters.md)
- [first-call summary measurements](wasm-first-call.md)

---

## 🚀 快速开始

### Reproduce a measurement

Use the repository's pinned Node/npm and locked dependencies. Build each selected
revision, then copy its `dist/`, `bridge/`, `scripts/`, `skills/`, `package.json`
and `package-lock.json` to a separate runtime directory before editing production
sources. That copy needs the matching locked dependencies. The initial frozen
baseline is `42b0ed126df7d5b908ac369715378d5e519051b7`.

The public artifacts and acquisition digests are in `corpus.json`. Supply their
actual local paths and an optional JSON object of explicit provider environment
settings. Do not add credentials to that file.

```sh
export NODE_OPTIONS=--max-old-space-size=512
export UV_THREADPOOL_SIZE=1
export GOMAXPROCS=1
export GOMEMLIMIT=2GiB
python3 experiments/analysis-efficiency/resource-run.py -- \
  node experiments/analysis-efficiency/measure.mjs \
  --runtime-root /absolute/frozen-runtime \
  --revision 42b0ed126df7d5b908ac369715378d5e519051b7 \
  --artifact obsidian.asar --path /absolute/obsidian-1.14.4.asar \
  --providers /absolute/selected-providers.json \
  --output /absolute/new-measurement-directory
```

The wrapper requires Linux cgroup v2 and a working user systemd manager. It owns
a unique transient scope, restricts the whole process tree to one CPU, caps
memory at 4 GiB with no swap and 192 tasks, and samples memory/tasks/CPU every
100 ms.

An actual Codex + Chromium trial exceeded the initial 192-task allowance and
was stopped automatically below 1 GiB sampled memory, with no OOM. The isolated
browser model retry uses `resource-run.py --task-ceiling 256 -- COMMAND...` to
account for both process trees.

Compare paired runs with the same runner source and provider settings:

```sh
node experiments/analysis-efficiency/compare.mjs \
  --left /absolute/complete-run --right /absolute/summary-run \
  --output /absolute/new-comparison.json
```

---

## ⚡ 功能

### Optimization priorities

Agent token consumption and end-to-end task time are the primary objectives.
Memory and CPU consumption are also measured and controlled. Correctness,
Evidence provenance, caller choices and existing-user compatibility are
acceptance conditions for every optimization.

- **Tokens**: minimize unnecessary model-visible data while returning the requested facts and useful provenance inline.
- **Time**: measure completion of the same analyst question, including startup, provider work, transport, client processing and model corrections.
- **Memory**: record aggregate process-tree peaks and limit/cleanup events.
- **CPU**: record aggregate CPU time for the completed task.

### What the numbers mean

`summary.json` records the selected artifact's before/after SHA-256, runtime
file-table digest, dispatcher and lockfile digests, runner source digest,
Node/SDK versions, catalog sizes, caller arguments and producer Evidence IDs.
`runtime-files.json` identifies the exact compiled files measured.

Every tool call measures three intervals at the public SDK Transport boundary:
- Client work before `send`, including first-use schema preparation.
- Transport/server roundtrip, including framing, I/O, server work and client JSON parsing.
- Client work after the parsed response reaches `onmessage`.

---

## ⚙️ 技术栈

- Node.js (pinned via .nvmrc)
- Python 3 (resource-run.py wrapper with cgroup v2)
- MCP SDK 2.3.1
- Ghidra (ephemeral databases)
- WABT (WASM analysis)
- Codex CLI (agent trials)

---

## 📊 指标

### Obsidian delivery checkpoint

The two recorded pairs compare frozen baseline `42b0ed12` and candidate `f1106181` in complete to summary and summary to complete order.

| Measurement                                         |        Complete | Summary + exact path view |
| --------------------------------------------------- | --------------: | ------------------------: |
| Tool-result compact JSON, including lifecycle calls |   937,404 bytes |              16,322 bytes |
| Tool calls per question                             |               3 |                         4 |
| Complete catalog compact JSON                       | 2,326,609 bytes |           2,370,505 bytes |
| Workflow elapsed, first pair                        |         11.04 s |                   10.23 s |
| Workflow elapsed, second pair                       |         10.57 s |                   10.18 s |

Reply JSON falls by about 98.3%; catalog JSON grows by about 1.9%.

### Same-question model and selected-source follow-up

Two actual SQLite model trials preserve all 12 requested facts. Calls change from 42 to 3, workflow time from 292.64 to 63.79 seconds, and reported input tokens from 304,698 to 74,387.

### Frozen runtime references

| Stage                                          | Source revision                            |
| ---------------------------------------------- | ------------------------------------------ |
| Original baseline                              | `42b0ed126df7d5b908ac369715378d5e519051b7` |
| Inventory delivery pairs                       | `f110618184e99f89fa776bc5e59703a61baa7c57` |
| Initial WASM pilot                             | `0002caa07ffd7319672a21ce901dbfbc55e21aac` |
| Exact WASM summary guidance                    | `259e99dd906290189217cf0fb1a19769bcf87101` |
| Main-integrated package/migration verification | `0eb42a1eb8ab17aed04240174cb77612e733a1d8` |

---

## 🏛️ 设计

### Optimization priorities and gradual migration

Migrate domain by domain with focused real-artifact checks and relevant shared
contract checks. The native follow-up verifies selected PE and DOS MZ entry facts;
DOS COM and complete native application behavior remain unverified.

Old clients using existing CLI/MCP requests against the candidate retain the
complete default path. New summary/view options require explicit selection.

### Task-scoped analyzer experiment

The recorded jq pair explicitly disables `Decompiler Switch Analysis` through `REA_GHIDRA_DISABLED_ANALYZERS`. One disabled/default pair takes 174.77/276.23 seconds. **Keep the production default.**

### Real Codex CLI trials

`agent-trial.py` accepts an explicit frozen runtime and revision, the same
question templates in `questions/`, and optional explicit provider settings.
Raw events and answers remain outside Git. A zero exit code is not a factual grade.

### CI attribution

| Failure                                  | Main `c22111eca`                                                                         | Reviewed PR `8d6f7eb1c`                                                                |
| ---------------------------------------- | ---------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------- |
| Empty prompt completion after open       | [main shard](https://github.com/morluto/rea/actions/runs/38224038323/job/114727940506)   | [PR shard](https://github.com/morluto/rea/actions/runs/38228724531/job/114742955092)   |
| Packaged prompt completion, Darwin arm64 | [main package](https://github.com/morluto/rea/actions/runs/38224038323/job/114728030045) | [PR package](https://github.com/morluto/rea/actions/runs/38228724531/job/114743092314) |
| Packaged prompt completion, Darwin x64   | [main package](https://github.com/morluto/rea/actions/runs/38224038323/job/114728029923) | [PR package](https://github.com/morluto/rea/actions/runs/38228724531/job/114743092079) |

---

## 📁 目录

「待补充」

---

## 🔗 资源

「待补充」