---
title: (翻)国外一篇很好的关于enable_if的介绍
date: 2019-04-30 23:33:04
updated: 2019-04-30 23:33:04
tags: [template, c++11, c++]
---

[原文链接](https://eli.thegreenplace.net/2014/sfinae-and-enable_if/)

## 背景
在C++中将函数重载与模板混合时，必须考虑一个有趣的问题。由于模板过于包容，以至于当与重载混合时，结果可能会令人惊讶：  
```C++
void foo(unsigned i) {
  std::cout << "unsigned " << i << "\n";
}

template <typename T>
void foo(const T& t) {
  std::cout << "template " << t << "\n";
}
```
当调用`foo(42)`时，打印的结果是`template 42`。因为42默认是一个`signed int`类型（42U则为unsigned int），当编译器检查重载时，第一个函数需要一次转换（int to unsigned int），而第二个模板则是完全匹配，所以它选择了第二个。  
<!-- more -->
*但是当第一个函数的类型为int时，函数重载则会选择第一个，因为在重载的决议中，非模板函数优先级高于模板*  
当编译器查看作为模板的重载候选项时，它必须实际执行将显式指定或推导类型替换为模板参数。这并不一定会产生敏感的代码，如下面的代码：  
```C++
int negate(int i) {
  return -i;
}

template <typename T>
typename T::value_type negate(const T& t) {
  return -T(t);
}
```
当调用`negate(42)`时，编译器会选择第一个函数并返回-42。然而，当寻找最完美的重载匹配时，所有的候选项都必须被考虑。当编译器考虑第二个模板`negate`时，它将推到类型（此例中为int）替入模板，然后就会得到下面的解释：  
```C++
int::value_type negate(const int& t);
```
上面这串代码是有问题的，因为`int`并没有叫做`value_type`的类型。但是编译器并不会报错，事实上，C++标准对于这样的情况有一个特别的条款，来准确解释编译器的行为。  
## SFINAE
在C++11标准的最新草稿中，14.8.2节中描述了相关的内容：  
当一个替代错误发生时（比如上面的例子中，对于某些特定类型的类型推断错误）,并不会产生错误。编译器会简单地忽略这个候选项而去查看其他的。  
在C++中，这个规则被称为“Substitution Failure Is Not An Error”，简称**“SFINAE”**  

> C++中的具体条款如下：
 If a substitution results in an invalid type or expression, type deduction fails. An invalid type or expression is one that would be ill-formed if written using the substituted arguments. Only invalid types and expressions in the immediate context of the function type and its template parameter types can result in a deduction failure.   
 如果一个替代会导致无效的类型或表达式，类型推断会错误。无效的类型或表达式为：使用替代参数之后会产生错误的组织。只有函数类型和它的模板参数类型的最近上下文中的无效类型或表达式会导致推断失败。  

然后列出可能被认为无效的可能方案，比如在限定名称中使用不是类或枚举类型的类型，尝试创建对void的引用等。  
关于上面提到的“最近上下文”，考虑下面这个例子：
```C++
template <typename T>
void negate(const T& t) {
  typename T::value_type n = -t();
}
```
如果上面这个重载匹配一些基本类型，会因为函数体内的`T::value_type`得到一个编译错误。因为这个超出了“函数类型和模板参数地最近上下文”*（这里的个人理解，这个问题出现在模板替换之后，而不是在模板替换之时，如在const T& t中或是在typename T中，结合下面`enable_if`的使用应该会容易理解一些）*。  
从上面这个例子中，我们可以了解到，如果我们要写一个只对某些类型有意义的模板，我们必须使它对于某些无效类型失败在类型推导中，因为这样只会产生推导失败，使得编译器忽视这个候选项而不是产生编译错误。  

## enable\_if ----编译时模板的选择
SFINAE被证明是有用的，以至于程序员在早期地C++中就明确使用这个。一个最值得关注的基于这个目的的工具就是`enable_if`，它的定义如下：
```C++
template <bool, typename T = void>
struct enable_if
{};

template <typename T>
struct enable_if<true, T> {
  typedef T type;
};
```
我们可以写出这样的代码：
```C++
template <class T,
         typename std::enable_if<std::is_integral<T>::value,
                                 T>::type* = nullptr>
void do_stuff(T& t) {
  std::cout << "do_stuff integral\n";
    // an implementation for integral types (int, char, unsigned, etc.)
}

template <class T,
          typename std::enable_if<std::is_class<T>::value,
                                  T>::type* = nullptr>
void do_stuff(T& t) {
    // an implementation for class types
}
```
注意在这里的SFINAE。当我们调用`do_stuff<int>`时，编译器会选择第一个重载,因为`std::is_integral<int>`为真，然后`struct enable_if`条件为真的特例被使用，因此它的内置`type`被设置为`int`。第二个重载被忽略，因为`std::is_class<int>`为假，使用的是`struct enable_if`条件为假时的特例，然后该特例没有变量`type`，所以会导致替换失败（而不会返回编译错误）。  
`enable_if`在Boost中很早就已经被使用，但是在标准C++库中，是从C++11才开始正式使用。在C++14中，为了使用方便，增加了新的别名如下：
```C++
template <bool B, typename T = void>
using enable_if_t = typename enable_if<B, T>::type;
```
使用`enable_if_t`，上面的例子可以改写为：
```C++
template <class T,
         typename std::enable_if_t<std::is_integral<T>::value>* = nullptr>
void do_stuff(T& t) {
    // an implementation for integral types (int, char, unsigned, etc.)
}

template <class T,
          typename std::enable_if_t<std::is_class<T>::value>* = nullptr>
void do_stuff(T& t) {
    // an implementation for class types
}
```
## enable_if的使用
`enable_if`是一个非常实用的工具，在C++11标准模板库中多次被提及。因为它是`type traits`（一种限制模板转换为那些具有特定属性的类型的方式）的核心部分。  
如果没有`enable_if`，当我们使用模板参数定义一个函数时，这个函数会去适配所有可能得类型。有了`type traits`和`enable_if`，我们可以针对不同的类型创建不同的函数。

> 这是在使用模板和重载的一种折中方式，C++又另外的工具可以在运行时实现类似的功能。但是`type traits`使我们在编译时就可以完成这项工作，而不必在运行时造成额外的开销。  

一个很有用的例子是构造具有两个参数的`std::vector` 
```C++
// Create the vector {8, 8, 8, 8}
std::vector<int> v1(4, 8);

// Create another vector {8, 8, 8, 8}
std::vector<int> v2(std::begin(v1), std::end(v1));

// Create the vector {1, 2, 3, 4}
int arr[] = {1, 2, 3, 4, 5, 6, 7};
std::vector<int> v3(arr, arr + 4);
```
上面使用了两种形式的双参数构造。忽略分配器，这些构造函数应该是这么定义的：
```C++
template <typename T>
class vector {
    vector(size_type n, const T val);

    template <class InputIterator>
    vector(InputIterator first, InputIterator last);

    ...
}
```
上面两个构造函数都有两个参数，但第二个函数具有模板的“catch-all”属性。尽管模板参数`InputIterator`有描述性的名字，但是它没有语义含义。编译器不会介意它被叫做什么。  
问题在于，即使是构造v1的时候，如果不做一些特别的工作，第二个构造函数依旧会被考虑。这是因为`4`的类型是`int`而不是`size_t`，所以去适配第一个函数时，编译器需要做一次类型转换，而第二个则是完全匹配。  
为了避免这个问题，使第二个函数仅被迭代器调用，这时候就需要用到`enable_if`。  
下面是第二个构造函数的实际定义：
```C++
template <class _InputIterator>
vector(_InputIterator __first,
       typename enable_if<__is_input_iterator<_InputIterator>::value &&
                          !__is_forward_iterator<_InputIterator>::value &&
                          ... more conditions ...
                          _InputIterator>::type __last);
```
这里使用了`enable_if`使其只有当输入为输入迭代器的时候才被使用。对于前向迭代器，有另外的重载，可以使执行效率更高。  
就像之前提到的，C++11标准库里用到了很多`enable_if`。比如`string::append`就用了和上面很像的一种方法。  
有一些不同的例子是`std::signbit`，被设计于定义所有的算数类型。下面是一个简单的关于它的定义版本（在cmath中可以找到）  
```C++
template <class T>
typename std::enable_if<std::is_arithmetic<T>, bool>::type
signbit(T x)
{
    // implementation
}
```
如果不使用`enable_if`，请考虑库实现者可能的实现： 一种是为每种已知算术类型重载函数。那非常冗长。另一种方法是使用不受限制的模板。但是，如果我们向它传递一个错误的类型，比如`std::string`时，我们很可能在使用时遇到一个相当模糊的错误。 使用`enable_if`，我们既不需要编写样板文件，也不会产生错误的错误消息。如果我们使用错误类型调用上面定义的`std::signbit`，我们将得到一个相当有用的错误，指出无法找到合适的函数。  

## 一种更加先进的enable_if版本
不可否认，`std::enable_if`是笨拙的，甚至`enable_if_t`都没有多大帮助，尽管它稍微简洁了一点点。我们仍然需要以一种经常模糊返回类型或参数类型的方式将它混合到函数声明中。这就是为什么网上的一些消息建议制作更高级的高级版本。就作者而言，他认为这是一个错误的权衡。  
`std::enable_if`是一个很少使用的构造。因此，减少冗长不会给我们带来太大的影响。另一方面，使它更神秘是有害的，因为每次我们看到它，我们必须考虑它是如何工作的。 本文显示的一些实现是非常简单的情况。 最后作者提出，C ++标准库使用了详细的“笨拙”版本的`std::enable_if`，而没有定义更复杂的版本。他觉得这是正确的决定。  


