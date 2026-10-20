---
title: Foundation XML Archive Encoding Fixtures
tags: [macos, foundation, xml, encoding, utf-16, bom]
category: testing
created: "2024-01-01"
updated: "2026-10-20"
source: 'external'
type: 'index'
status: 'stable'
lifecycle: 'active'
---

# Foundation XML Archive Encoding Fixtures

## 💡 概述

`producer.swift` creates an actual NSKeyedArchiver XML representation, then
uses Foundation string encoding to produce BOM-marked UTF-16 little- and
big-endian representations of the same XML graph. Foundation's
PropertyListSerialization reads both representations successfully; `plutil
-lint` also accepts all three files.

---

## 🚀 快速开始

```sh
xcrun swiftc producer.swift -o writer
./writer archive.xml little-endian.plist big-endian.plist
plutil -lint archive.xml little-endian.plist big-endian.plist
```

---

## 🏛️ 设计

Only the UTF-8 XML source capture is committed. The tests encode its same text
as UTF-16 so no compiled binaries or binary fixtures are committed. This is
format/filesystem verification; no archived classes are instantiated by REA.