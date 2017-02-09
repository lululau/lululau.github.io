---
layout: post
title:  "搜索 Evernote 笔记的小技巧"
date:   2014-01-04 02:18:00
categories: 
  - 'Mac-App'
---

#### 1. 只搜索笔记的标题

如果想只搜索标题中包含某些关键字的笔记，例如，只搜索标题中包含 `Ruby` 的关键字，而不管笔记内容是否包含这个关键字，那么可以在 Evernote.app 的搜索框中输入： `intitle:Ruby`。

如果使用 Alfred.app 的 Evernote 工作流插件，那么可以在 Alfred 中键入 `ent 关键字` 来完成只对笔记标题进行匹配的搜索。

#### 2. 精确搜索

如果在 Evernote 的搜索框中搜索 `Programming Ruby`，那么会搜索出所有包含了 `Programming` 关键词和所有包含了 `Ruby` 关键词的笔记，如果本意是精确地搜索包含 `Programming Ruby` 这个完整的字符串的笔记，那么把这个字符串用双引号括起来即可：`"Programming Ruby"`，在 Alfred Evernote 插件中同样也可以使用双引号将关键字括起，来达到同样的目的。