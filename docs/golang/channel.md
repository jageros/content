---
type: "posts"
author: "jager"
title: "channel注意事项"
date: "2021-08-03"
tags: [
"go",
"channel",
]
---

> channel是go语言编程中必不可少的元素，但对不同状态的channel作read，write和close操作会有不同的结果，
> 稍有操作不当，将导致系统 panic，所以本文介绍了channel不同状态的操作结果

<!--more-->

## channel不同状态的操作结果：
操作|channel状态|结果
-|-|-
Read|nil|阻塞
||打开且非空|输出值
||打开但空|阻塞
||关闭的|<默认值>, false
||只写|编译错误
|-|
Write|nil|阻塞
||打开的但填满|阻塞
||打开的且不满|写入值
||关闭的|panic
||只读|编译错误
|-|
close|nil|panic
||打开且非空|关闭channel；读取成功，直到通道耗尽，然后读取产生的默认值
||打开但空|关闭channel；读到生产者的默认值
||关闭的|panic
||只读|编译错误
