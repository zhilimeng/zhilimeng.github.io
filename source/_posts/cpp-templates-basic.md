---
title: cpp templates basic
date: 2019-04-20 14:29:45
tags: [C++11]
categories: C++
---
## C++模板基础
C++模板可以让我们写出类型无关的代码，极大地提高了代码的适用性，C++标准库的几乎所有代码都是模板代码。
### 函数模板
函数模板可为不同的类型提供函数行为，其定义与普通函数类型，只不过类型是参数化的，以关键字`typename`开头，模板参数列表以逗号分开，例如：
```
template<typename T>
T max(T a,T b)
{
  return b < a ? a : b;
}
template<typename T1,typename T2>
auto max(T1 a, T2 b)
{
  return b < a ? a : b;
}
```
### 模板编译
用实际类型替换模板参数的过程称为实例化，结果是模板的一个实例。注意，仅仅使用函数模板就会触发实例化过程。尝试实例化一个不支持内部所有操作的类型将会导致编译期错误，例如：
```
std::complex<float> c1,c2; // doesn't provide operator <
...
::max(c1,c2); // error at compile time
```
模板在两个阶段编译   
1. 在定义时没有实例化(`definition time`)，模板代码进行正确性检测，忽略模板参数，这个过程包括：
- 句法错误，例如缺少分号
- 使用不依赖模板参数的未知的命名（类型名，函数名）
- 不依赖模板参数的静态断言
2. 实例化时间(`instantiation time`),模板代码会再次进行检测确保所有代码是有效的，特别地，所有依赖模板参数的部分都会被重新检查   
事实上，一些编译器不会执行第一阶段的所有检查，所以一些错误直到第一次实例化时才会被发现。编译器在编译的时候需要看到模板的定义，这与普通函数的编译链接不同（函数声明对于编译就足够了）。处理这个问题的一个方法是将实现放在头文件中。
### 模板参数推断
模板参数由我们传递的参数决定，如果传递两个`ints`到参数类型`T`。编译器就得出`T`一定是`int`类型。在类型推断的时候自动类型转换是有限制的：
- 如果通过引用声明调用参数，不允许进行自动类型转换，每个类型T都必须正确地匹配
- 如果通过传值声明调用参数，`const` 或者`volatile`限定会被忽略，引用转换为引用的类型，原始数组或函数转换为相应的指针类型。
```
template<typename T>
T max(T a, T b)
{
  return b < a ? a : b;
}
const int c = 42;
int i = 10;
max(i,c); // T is deduced as int
max(c,c); // T is deduced as int
int &ir = i;
max(i,ir); // T is deduced as int
int arr[4];
max(&i,arr); // T is deduced as int*
```
### 返回类型推断
最简单的方式是使用`auto`最为返回类型，但是要求实际返回类型能够从函数体内的声明推断。C++ 11可以使用关键字`decltype`，
```
template<typename T1, typename T2>
auto max(T1 a, T2 b)-> decltype(b<a?a:b)
{
  return b < a ? a : b;
}
```
但是这种方式有一个缺点，当`T`是引用时，返回类型也是引用类型，这种情况下，应该返回`T`的退化类型，`auto`关键字初始化的类型始终是退化的。
```
#include <type_traits>
template<typename T1,typename T2>
auto max(T1 a, T2 b) -> typename
std::decay<decltype(true?a:b)>::type
{
  return b < a ? a : b;
}
```
C++11，标准库提供了`std::common_type<>::type`可以得到两个不同类型模板参数的公共类，注意，`std::common_type<>`也是退化的。
```
#include <type_traits>
template<typename T1,typename T2>
std::common_type<T1,T2> max(T1 a, T2 b)
{
  return b < a ? a : b;
}
```
在内部实现，会根据操作符`?:`的语言规则和特定类型的特化来选择最终的类型，`::max(4,7.2)`和`::max(7.2,4)`得到的类型都是`double`。
### 默认模板参数
你可以为模板参数定义默认值，例如，如果你想结合定义返回类型和具有多个参数类型的方法，你可以引入一个模板参数`RT`作为两个参数公共类型的返回类型。
1. 可以使用操作符`?:`
```
#include <type_traits>
template<typename T1,typename T2,
         typename RT = std::decay_t<decltype(true?T1():T2())>>
RT max(T1 a, T2 b)
{
  return b < a ? a : b;
}
```
2. 使用`std::common_type<>`声明返回类型的默认值
```
#include <type_traits>
template<typename T1, typename T2,
         typename RT = std::common_type_t<T1,T2>>
RT max(T1 a, T2 b)
{
  return b < a ? a : b;
}
```
作为调用者，你可以使用默认值作为返回类型或者显示声明返回类型
```
auto a = ::max(4,7.2);
auto b = ::max<double,int,long double>(7.2,4);
```
但是，现在必须声明三个类型才能声明返回类型。相反，我们需要返回类型作为第一个模板参数，这也是可能的。
```
template<typename RT = long, typename T1, typename T2>
RT max(T1 a, T2 b)
{
  return b < a ? a : b;
}
```
### 重载函数模板
和普通函数一样，函数模板也可以重载，例如：
```
// maximum of two int values:
int max(int a, int b)
{
  return b < a ? a : b;
}
// maximum of  two values of any type:
tempalte<typename T>
T max(T a, T b)
{
  return b < a ? a : b;
}
int main()
{
  ::max(7,42);// calls the nontemplate for two ints
  ::max(7.0,42.0); // calls max<double>
  ::max('a','b'); // calls max<char>
  ::max<>(7,42); // calls max<int>
  ::max<double>(7,42); // calls max<double>
  ::max('a',42.7); // calls the nontemplate for two ints
}
```
有相同名字的非模板函数和能实例化该类型的模板函数是可以共存的，非模板函数的优先级高于由模板函数推断的。但是如果模板可以产生一个更好的匹配，就会选择模板函数，也可以显示地说明使用模板。
### 总结
- 函数模板为不同模板参数定义了一个函数家族
- 当你根据模板参数向函数参数传递变量对象时，函数模板会根据相应的参数类型实例化模板参数
- 你可以显示地声明模板参数
- 可以定义默认模板参数
- 可以重载函数模板，但需要确保对于任何调用只有一个匹配
- 在你调用之前确保编译器看到所有重载的函数模板版本。
