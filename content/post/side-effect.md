---
title: "Haskell中Monad是如何解决side effect的？"
date: 2017-07-19T15:34:45+08:00
draft: false
tags: [Haskell]
---

> 翻译自 *[Why are side-effects modeled as monads in Haskell?](https://stackoverflow.com/a/2488852)*

假设一个函数有side effects。如果将函数导致的side effect包在函数的输入和输出中，则函数又变成了pure的。

所以对于一个impure的函数`f'`
```hashell
f' :: Int -> Int
```

通过添加**`RealWorld`**来将impure函数变成pure的函数。

```haskell
f :: Int -> RealWorld -> (Int, RealWorld)
-- 将RealWorld作为一个参数，并且将修改之后的RealWorld返回
```

这样的话，`f`现在就是一个pure的函数。为了方便，记`IO a = RealWorld -> (a, RealWorld)`，`f`的类型就可以简写为
```haskell
f :: Int -> IO Int
```

**`IO`是对RealWorld的一种封装，避免了程序员直接操作RealWorld对象。对于程序员来说，直接操作RealWorld对象是很危险的一件事。例如，如果我们获取了一个RealWorld类型的值，我们可能拷贝这个值。从逻辑上讲，这种行为是不可能的。**

```haskell
getLine :: IO String = RealWorld -> (String, RealWorld)
getContents:: String -> IO String = String -> RealWorld -> (String, RealWorld)
putStrLn :: String -> IO () = String -> RealWorld -> ((), RealWorld)
```

假设现在我们想从console读一个文件名，然后读取文件，最后将文件内容输出。我们应该如何通过上述的pure函数实现呢？

```haskell
printFile :: RealWorld -> (String, RealWorld)
printFile world0 = let (filename, world1) = getLine world0
                       (contents, world2) = getContents filename world1
                   in  (putStrLn contents) world2 -- results in ((), world3) 
```

从上述示例中可以发现函数调用的模式：

```haskell
...
(<result-of-f>, worldY) = f worldX
(<result-of-g>, worldZ) = g <result-of-f> worldY
...
```

所以我们可以通过定义`~~~`操作符来绑定一系列的函数：
```haskell
(~~~) :: (IO b) -> (b -> IO c) -> Io c

(~~~) ::    (RealWorld -> (b, RealWorld)) 
        ->  (b -> RealWorld -> (c, RealWorld))
        ->  (RealWorld -> (c, RealWorld))

(f ~~~ g) = let (resF, worldY) = f worldX in
                g resF worldY
```

上述的`printFile`函数可以简写为：
```
printFile = getLine ~~~ getContents ~~~ putStrLn
```
可以发现`printFile`并没有显示地操作`RealWorld`。

---

如果现在我们想将文件的内容都转换成大写字母。由于`upperCase`是一个pure函数，类型如下：
```haskell
upperCase :: String -> String
```

为了让它应用到real world中，它不得不返回`IO String`，因此我们可以将`upperCase`提升为：
```haskell
impureUpperCase :: String -> RealWorld -> (String, RealWorld)
impureUpperCase str world = (upperCase str, world)
```

这种方法具有一般性，即将一个pure的函数转换成impure：
```haskell
impurify :: a -> IO a
impurify :: a -> RealWorld -> (a, RealWorld)
impurify a world = (a, world)
```

所以`impureUppercase = impurify . upperCase`并且`printUpperCaseFile = getLine ~~~ getContents ~~~ (impurify . upperCase) ~~~ putStrLn`

---

回头看一下我们完成了什么。

1. 定义了操作符`~~~::IO b -> (b -> IO c) -> IO c`将两个impure函数串联
2. 定义了函数`impurify :: a -> IO a`将一个pure的值转换成impure的值

是不是很眼熟，`(>>=) = (~~~)`，`return = impurify`。我们就这样构造出了monad。