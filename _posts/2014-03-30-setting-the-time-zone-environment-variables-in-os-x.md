---
layout: post
title:  "在 OS X 中设置时区环境变量"
date:   2014-03-30 23:58:00
categories: 
  - Shell
  - 'Mac-OS-X'
---

在 Linux 和 OS X 中，为进程设置不同的时区都可以通过修改 `TZ` 这个环境变量来实现，在 OS X 上使用这样的方法来为进程设置不同的时区：

```ruby
export TZ=Asia/Shanghai
ruby -e 'puts Time.now'  # => 显示上海时间
export TZ=Asia/Tokyo
ruby -e 'puts Time.now'  # => 显示东京时间
TZ=America/Los_Angeles ruby -e 'puts Time.now'  # => 显示洛杉矶时间
```

那么这些时区的取值从哪里获取呢？

```sh
sudo systemsetup -listtimezones
```

另外也可以通过 `systemsetup` 命令来获取当前的时区设置：

```sh
sudo systemsetup -gettimezone
```

设置时区：

```sh
sudo systemsetup -settimezone Europe/Berlin  # 将当前时区设置为柏林时间
```
