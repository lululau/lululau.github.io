---
layout: post
title:  "从一个几十GB上百GB的日志文件中过滤所关心的错误信息"
date:   2013-11-22 14:27:00
categories: 
  - Shell
---

先根据故障发生的时间定位关键的日志在整个日志文件中的大致位置，然后用 `dd` 输出这部分的日志再 `grep` 即可:

```sh
# 显示一个文件中从第 50G 个字节开始的长度为 1K 字节的内容
offset=$((50*1024*1024*1024-1))
length=1024
dd if=file_name bs=1 skip="$offset" count="$length" | grep regex
```