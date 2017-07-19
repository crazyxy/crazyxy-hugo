---
title: "Haskell中Monad是如何解决side effect的？"
date: 2017-07-19T15:34:45+08:00
draft: false
tags: [Haskell]
---

> 翻译自 *[Why are side-effects modeled as monads in Haskell?](https://stackoverflow.com/a/2488852)*

假设一个函数有side effects。如果将函数导致的side effect包在函数的输入和输出中，则函数又变成了pure的。

