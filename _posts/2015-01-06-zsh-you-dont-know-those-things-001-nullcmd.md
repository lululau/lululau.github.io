---
layout: post
title:  "zsh 你不知道那些事儿-003-NULLCMD"
date:   2015-01-06 23:30:00
categories: 
  - Shell
---

几乎每一本 Linux / Shell 入门的书中都会讲到如何新建一个空文件，那就是 `touch` 命令：

```sh
touch file_name
```

实际上在 bash 中有更高效的方法（其实就是少敲 4 个字符，😄）：

```sh
> file_name
```

但是，如果在 zsh 中尝试执行以上命令，会发现 zsh 陷入到一个进程中不会退出，直到按 `Ctrl-d`为止。这是因为在 zsh 中执行一个只有IO重定向而没有命令名字的 command line 时，zsh 会使用变量 `NULLCMD` 的值作为这个 command line 的命令名字，而 `NULLCMD` 的默认值为：`cat`，就是说当在 zsh 中执行 `> file_name` 时，实际上执行的是 `cat > file_name`，如果要获得和 bash 中一样的体验，可以将 `NULLCMD` 的值修改为 `:`：

```sh
export NULLCMD=:
```

可以将以上代码加入到 zsh 配置文件中。

类似的，当在 zsh 中执行 `< file_name` 这条命令的时候，zsh 会使用变量 `READNULLCMD` 的值作为这行 command line 的命令名字， `READNULLCMD` 的默认值为：`more`。

PS:

(1). `touch` 命令

实际上 `touch`命令的主要功能并不是用来创建一个新的空文件。它的手册（`man 1 touch`）中的说明是： 

> change file access and modification times

(2). `:` 命令

是的，`:` 是一个命令的名字，bash 和 zsh 都内建了这个命令，它的功能非常简单，就是：什么都不做，退出状态码为 0。