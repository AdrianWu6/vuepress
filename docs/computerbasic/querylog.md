---
title: Linux下查询日志的技巧
date: 2025-05-23
categories:
- OS
tags:
- Shell
---

## 1.直接将日志从服务器上拿下来

ftp，sftp将log文件拿下来后，notepad++打开查询

## 2. vi编辑器

vi log.log

按下`/`键进入搜索模式，在`/`后输入想输入的关键字，按n键查找下一处。

## 3. grep查找字段

```
# 展示出所有带关键字的那一行
grep 关键字 log.log
# 查看关键字后50行的内容
grep -A 50 关键字 a.log
# 查看关键字后50行的内容 增强版 通过翻页查询
grep -A 50 关键字 a.log | less
pageup pagedown 前后翻页 而且也可以通过 / 进行搜索
按G直接翻到当前文档结尾
# 查看前后50行的内容
grep -C 50 关键字 a.log|less
```

## 4.实时监测含关键字的日志

```
tail -f log.log|grep -A 50 关键字 
```

## 5.统计报错的条数

```
grep -c 关键字 log.log
```


