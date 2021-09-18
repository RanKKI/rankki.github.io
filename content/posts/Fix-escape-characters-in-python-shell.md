---
title: 修复在 Python 交互界面中无法使用方向键和 tab 键
date: 2019-03-23 14:32:12
tags: 
 - Python
categories:
 - 技术
---

## 背景

在日常使用 Python 中（使用交互界面）时，经常遇到按方向键，会出现^[[D, ^[[A 等符号，以及不能使用 tab 自动补全，

查了下似乎是 readline 这个 lib 的问题， 用 brew info 查了下， 我的 readline 库是 8.0.0 版本

```bash
$ ~ brew info readline
  readline: stable 8.0.0 (bottled) [keg-only]
  Library for command-line editing
  https://tiswww.case.edu/php/chet/readline/rltop.html
  /usr/local/Cellar/readline/8.0.0 (50 files, 1.5MB)
  Poured from bottle on 2019-03-04 at 12:08:49
  From: https://github.com/Homebrew/homebrew-core/blob/master/Formula/readline.rb
```

说是更换下库版本就行，但是怕因此影响到其他东西，于是就没有搞， 后来再查资料发现了解决办法，

## 解决方法

方法来源：[StackOverflow](https://stackoverflow.com/a/41560355)

在 `/usr/local/opt/readline/lib/` 中，我的只有 `libreadline.8.dylib`

只要把这个链接到 `libreadline.6.dylib` 和 `libreadline.7.dylib` 就能解决问题了。