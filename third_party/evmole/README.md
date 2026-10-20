---
title: EVM Interface Upstream Profile
tags: [evm, evmole, ethereum, npm, wasm, smart-contract]
category: third-party
created: "2024-01-01"
updated: "2026-10-20"
source: 'external'
type: 'index'
status: 'stable'
lifecycle: 'active'
---

# EVM Interface Upstream Profile

## 💡 概述

REA integrates unchanged [EVMole](https://github.com/cdump/evmole) as the exact
npm dependency `evmole@0.9.3`, with lockfile integrity verification. Tag `0.9.3`
resolves to commit `0409707c2efef11a9043f9b4505cad5719054a7e`.

---

## 🏛️ 设计

The JavaScript binding and WASM engine remain in the original upstream package;
REA contains only carrier validation, a worker adapter and portable contracts.
No submodule or vendored engine fork is required. No signature database,
LLM decompiler, blockchain node or RPC dependency is acquired.

Upstream is MIT, copyright (c) 2023 Maxim Andreev. Its npm tarball omits the
license file, so REA packages the unchanged tag's [LICENSE](LICENSE) here.
The engine is initialized through the upstream `no_tla`/`initSync` interface
only inside an owned worker; importing REA services performs no WASM/file I/O.

---

## ⚡ 功能

Initial interface analysis uses `contractInfo` with selectors, arguments and
stateMutability enabled. Selector recovery also extracts/reports terminal metadata;
its reported representation is retained with explicit unknown precision/coverage.
Other upstream outputs are separate future contracts;
in particular, CFG destination IDs are not byte addresses and terminal CBOR
metadata cannot establish a lossless integer claim through the current binding.