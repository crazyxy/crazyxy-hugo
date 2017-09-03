---
title: "is_callable implementation in C++"
date: 2017-09-03T21:38:48+08:00
draft: false
---

今天在看一个[C++11并发异步访问](https://github.com/crazyxy/asyncplusplus)的时候看到了下面一段代码：

```c++
typedef char one[1];
typedef char two[2];
template<typename Func, typename... Args, typename = decltype(std::declval<Func>()(std::declval<Args>()...))>
two& is_callable_helper(int);
template<typename Func, typename... Args>
one& is_callable_helper(...);
template<typename T>
struct is_callable;
template<typename Func, typename... Args>
struct is_callable<Func(Args...)>: public std::integral_constant<bool, sizeof(is_callable_helper<Func, Args...>(0)) - 1> {};
```

从函数的命名来看，`is_callable`是一个模版类，用来判断判断一个`Func`类型的函数能否使用参数`Args`进行调用，例如：

```c++
is_callable(function<void(int)>(int))::value; // true
is_callable(function<void(int)>(string))::value; //false
```

这篇博客讨论如何实现`is_callable`。

## C++ Member Detector

在[这篇](https://en.wikibooks.org/wiki/More_C++_Idioms/Member_Detector)Wiki中讨论了如何实现`Member Detector`。通过运用**Substitution Failure Is Not An Error (SFINAE)**，重载一个测试的函数，通过最终的选择测试结果来判断是否存在某个属性。例如：

```c++
template<typename T>
class DetectX {
    using Yes = char[2];
    using No = char[1];

    struct Fallback { int X; }; 
    struct Derived : T, Fallback { };

    template<typename U> 
    static No & func(decltype(U::X) *);
    
    template<typename U> 
    static Yes & func(U*);

  public:
    static constexpr bool RESULT = sizeof(test<Derived>(nullptr)) == sizeof(Yes);
};
```

上面这段代码检查类型`T`中是否存在成员`X`，注意，这里无论`X`是什么类型。在类中定义了两个类型，`Yes`和`No`，作为返回类型用来区别最终选择的函数的结果。`Fallback`类型在结构体内定义了一个名为`X`的成员变量，而类`Derived`则继承自`T`和`Fallback`。如果类型`T`中存在名为`X`的成员变量，`Derived`类型中会有两个名为`X`的成员，`decltype(U::X)`将无法调用，而会选择调用第二个函数，如果`T`中不存在名为`X`的成员变量，则会成功调用第一个函数。最终痛过返回值判断类型`T`中是否存在名为`X`的成员变量。

## is_callable

回到最初的那段代码，代码中同样定义了两个用来标记是或者否的类型，分别是`one`和`two`。跟`Member Detector`一样，同样定义了两个重载的函数，分别是：
```c++
template<typename Func, typename... Args, typename = decltype(std::declval<Func>()(std::declval<Args>()...))>
two& is_callable_helper(int);
template<typename Func, typename... Args>
one& is_callable_helper(...);
```
这两个函数的区别在于第一个函数多了一个默认的类型参数，即最后那一串`decltype(std::declval<Func>()(std::declval<Args>()...))`，如果前面的参数满足最后这个条件，则调用第一个函数，否则调用第二个函数。而实际上后面这个类型就是使用`Args`作为参数调用`Func`的返回类型，如果`Args`可以作为参数调用`Func`，则最后一个参数可以进行类型推导，从而调用第一个参数；否则调用第二个函数。代码最后通过继承`integral_constant`接口对结果进行的封装。`integral_constant`的第一个是类型`bool`，第二个是`bool`型的变量，通过调用`is_callable_helper<Func, Args...>(0)`来进行测试。