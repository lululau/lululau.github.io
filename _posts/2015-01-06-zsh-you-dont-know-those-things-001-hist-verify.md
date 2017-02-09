---
layout: post
title:  "zsh 你不知道那些事儿-001-HIST_VERIFY"
date:   2015-01-06 22:55:00
categories: 
  - Shell
---

bash 和 zsh 都有一套命令历史机制，历史替换(History Expansion)是其中的一部分, 例如：  

执行上一条命令：

```sh
!!
```

执行上上一条命令：

```sh
!-2
```

将上一条命令中的 foo 替换成 bar，然后再执行：

```sh
^foo^bar
```

在 bash 中，当你在 command line 中键入 `!!`，`!-2`，`^foo^bar` 这些命令然后回车的时候，对应的命令便会立即执行，zsh 的默认行为也是如此。但是 zsh 中可以通过设置 `HIST_VERIFY` 选项，让 zsh 只将历史替换展开，并不立即执行，用户确认没有问题后自行按回车键执行。

设置 `HIST_VERIFY` 选项的命令为：

```sh
set -o histverify
```

或

```sh
setopt histverify
```

如果使用 oh-my-zsh，那么这个选项已经打开（`${OH_MY_ZSH_ROOT}/lib/history.zsh`）