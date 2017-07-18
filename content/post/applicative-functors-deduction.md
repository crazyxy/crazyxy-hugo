---
title: "Applicative Functors Deduction"
date: 2017-07-18T14:52:52+08:00
draft: true
tags: [Haskell]
---

## 背景简介

在Haskell中，Functor和Applicative Functor的定义如下：

```haskell
class Functor f where
    fmap :: (a -> b) -> f a -> f b

class (Functor f) => Applicative f where
    pure :: a -> f a
    <*>  :: f (a -> b) -> f a -> f b
```

从上面的定义中可以看出，Functor和Applicative的instance都是type constructor，因为它需要吃一个类型`a`。许多type constructor都是Functor和Applicative的instance。例如`Maybe`，`[]`和`(->)r`。

在上面三个类型中`(->)r`最为怪异。它是一个function，并且一个吃了一个类型，只需要另外一个类型便可以construct一个新的类型。`(->)r`的`fmap`，`pure`和`<*>`的实现如下：

```haskell
instance Functor ((->)r) where
    fmap f g = \x -> f (g x)
```


```haskell
expr = (+) <$> (+3) <*> (*100) $ 5
```

## 类型推导


## 求值