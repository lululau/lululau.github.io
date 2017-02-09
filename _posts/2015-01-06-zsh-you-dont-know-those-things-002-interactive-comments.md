---
layout: post
title:  "zsh 你不知道那些事儿-002-INTERACTIVE_COMMENTS"
date:   2015-01-06 23:20:00
categories: 
  - Shell
---

在使用 bash 或 zsh 编写脚本的时候，可以使用 `#` 注释掉一行代码，在交互式的 shell command line 中我也习惯使用 `#` 来注释掉一条暂时不需要执行，但是接下来很可能会再次用到的命令（这样做的目的是当再次需要使用这条命令的时候，在命令历史中不用向上翻阅太多行记录）。在交互式的 zsh 中，默认是不能用 `#` 来注释一行命令的。解决方法是：打开 `INTERACTIVE_COMMENTS` 选项：

```sh
set -o interactivecomments
```

或

```sh
setopt interactivecomments
```

可以将以上代码加入到 zsh 配置文件中。