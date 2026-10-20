---
title: Source-Built Mach-O Header Captures
tags: [macos, mach-o, header, otool, fixture]
category: testing
created: "2024-01-01"
updated: "2026-10-20"
source: 'external'
type: 'index'
status: 'stable'
lifecycle: 'active'
---

# Source-Built Mach-O Header Captures

## 💡 概述

Captured on macOS arm64 with the host Xcode tools from the adjacent `fixture.c`:

```sh
xcrun clang fixture.c -o fixture
xcrun otool -l fixture > otool-load.txt
{ xcrun otool -h fixture; xcrun otool -l fixture; } > otool-header-load.txt
```

Only the temporary fixture path was replaced with `/owned/fixture`; tool fields
are unchanged. The first output contains load commands without a Mach header.
The second contains the real numeric header followed by those same commands.
These are format regression captures, not Hopper or Ghidra runtime proof.