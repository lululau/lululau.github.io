---
layout: post
title:  "使用 Bundler 安装的 Github gem 的安装位置"
date:   2013-09-28 22:45:00
categories: 
  - Ruby
---

```sh
echo $(gem environ | sed -n '/GEM PATHS/{n;s#.*- ##;p;}')/bundler/gems
```