---
title: "Applicative Functors Deduction"
date: 2017-07-18T14:52:52+08:00
draft: false
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
    fmap f g = \r -> f (g r)
```
`(->)r`的`fmap`函数实际上相当于`.`，因为`fmap`的类型为：
```haskell
fmap :: (a -> b) -> (r -> a) -> (r -> b)

```
因此对于`(->)r`Functor，`fmap = .`。`(->)r`的`pure`和`<*>`的实现为：

```haskell
instance Applicative ((->)r) where
    pure x = \_ -> x
    <*> f g = \r -> f r (g r)
```

接下来解释下面表达式的求值与类型推导
```haskell
(+) <$> (+3) <*> (*100) $ 5
```

## 求值

```haskell
  (+) <$> (+3) <*> (*100) $ 5
= (\x -> (+) (+ 3 x)) <*> (* 100) $ 5
= (\y -> (+) (+ 3 y) (* 100 y)) $ 5
= (+) (+ 3 5) (* 100 5)
= 508
```

## 类型推导

- Step 1

```haskell
let a = (+) <$> (+3)
```

`<$>`的类型是infix的`fmap`，因此`<$>`的两个参数类型分别为`(a->b)`和`f a`，结果为`f b`。在上面的表达式中第一个参数的类型为`Num a => a -> a -> a`，可以通过curry，将类型看成`Num a => a->(a->a)`。因此`a`的最终类型是`Num a => (->) a (a -> a)`。

- Step 2

```haskell
let b = a <*> (*100)
```

`<*>`的两边的参数类型分别是`f (a->b)`和`f a`。在`b`中，`<*>`左边表达式的类型是`Num a => (->) a (a->a)`，右边表达式的类型是`Num a => (->) a a`。因此`b`的类型为`Num a => (->) a a`。

- Step 3

```haskell
let c = b $ 5
```

`b`的类型为`Num a => a -> a`，因此`c`的类型为`int`。