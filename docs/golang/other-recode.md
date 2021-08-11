---
type: "posts"
author: "jager"
title: "零碎笔记"
date: "2021-08-03"
tags: [
"go",
]
---

> 本文记录一些golang程序的一些简短的要点

<!--more-->

## 求int64的最大值
```
maxInt64 := int64(^uint64(0)>>1)
```