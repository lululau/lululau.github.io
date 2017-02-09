---
layout: post
title:  "解决Mac外接显示器字体模糊的问题"
date:   2012-01-04 20:30:00
categories: 
  - Mac
  - Mac-OS-X
---

Mac外接显示器时，除非接的是Apple自家的显示器“ACD”，不然一般会遇到字体模糊发虚的问题。在终端中执行命令:

```sh
defaults -currentHost write -globalDomain AppleFontSmoothing -int 2
```

可以使用1到3作为该命令的最后一个参数，表示字体平滑渲染的强度。如果要恢复默认设置：

```sh
defaults -currentHost delete -globalDomain AppleFontSmoothing
```