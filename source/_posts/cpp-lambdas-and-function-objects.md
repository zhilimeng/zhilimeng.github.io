---
title: C++11 Lambda表达式和函数对象
date: 2018-11-25 00:10:33
tags: [C++11]
categories: C++
---
## Lambda表达式
c++11引入了lambdas表达式，可以用作参数或者一个局部对象。可以把它当作一个内联函数。最简单的lambda函数可以没有参数，例如：
```c++
[]{
  std::cout << "hello lambda" << std::endl;
}
```
<!-- more -->
### lambda语法
lambda表达式总是以引导符号`[]`开始，里边可以指定*capture*捕获非静态外部变量,在lambda引导符号`[]`和函数体`{}`之间，可以指定参数`mutable`、异常说明、属性设定和返回类型，这些都是可选项，因此，完成的Lambda表达式如下：
```
[...]  (...)  mutable throw ->retType {...}
```
#### 捕获外部变量
- [=] 表示外部变量通过值拷贝方式传递给lambda，lambda函数体不会改变外部变量的值
- [&] 表示外部变量通过引用方式传递给lambda，lambda函数体对外部变量具有写入权限
你可以对每个对象分别指定访问方式,例如：
```c++
int x = 0;
int y = 42;
auto qqq = [x,&y]{
          std::cout << "x: " << x << std::endl;
          std::cout << "y: " << y << std::endl;
          ++y; // OK
};
x = y = 77;
qqq();
qqq();
std::cout << "final y: " << y << std::endl;
```
输出：
```
x:0
y:77
x:0
y:78
final y: 79
```
使用`mutable`关键字可以在函数体内修改按值传递的外部变量的值，例如：
```c++
int id =0;
auto f = [id]() mutable{
  std::cout << "id: " << id << std::endl;
  ++id; // OK};
id = 42;
f();
f();
f();
std::cout << id << std::endl;
```
输出：
```
id: 0
id: 1
id: 2
42
```
#### 参数列表
和其他函数一样，lambda可以在圆括号内指定参数，注意，lambdas不能是模板，必须指定所有类型。例如：
```c++
auto l = [](const std::string& s){std::cout << s << std::endl};
l("hello lambda");  //prints "hello lambda"
```
#### 返回类型
返回类型在`->`后指定，如果没有指定返回类型，lambda从返回值推断。例如：
```c++
[]{return 42;}
```
返回类型为`int`。
```c++
[]()->double{return 42;}
```
返回类型为`double`。
#### 使用lambda
1. 使用lambda搜索集合中第一个值在x和y之间的元素
~~~c++
#include <algorithm>
#include <deque>
#include <iostream>
using namespace std;
int main()
{
  deque<int> coll = {1,3,19,5,13,7,11,2,17};
  int x = 5,y = 12;
  auto pose = find_if(coll.begin(),coll.end(), [=](int i){
    return i > x && i < y;
  });
  std::cout << "first elem > 5 and < 12: " << *pose << endl;
}
~~~   
2. 使用lambda作为搜索条件
```c++
#include <algorithm>
#include <vector>
#include <string>
#include <iostream>
using namespace std;
int main()
{
  std::vector<string> coll{"2","1","12","23","14","10"};
  std::sort(coll.begin(),coll.end(),
          [](const string& s1, const string& s2){
            if(s1.size() == s2.size())
              return s1 < s2;
            else
              return s1.size() < s2.size();
          })
}
```
总结：lambdas提供了一个方便、可读、快速、可维护的使用STL算法的方式。

## 函数对象
anything behaves like a function is a funciton，如果一个类提供了函数调用操作符，那么这个类的对象就可以作为函数对象。例如：
```c++
// simple function object that prints the passed argument
class PrintInt{
public:
  void operator()(int elem) const{
    cout << elem << " ";
  }
};
int main()
{
  vector<int> coll{1,2,3,4,5,6,7};
  for_each(coll.begin(),coll.end(),PrintInt());
  cout << endl;
}
```
### 使用函数对象的好处
1. 函数对象是具有状态的：函数对象可以有其他的成员函数和属性，实际上，由同一类型的两个不同函数对象表示的相同功能，在同一时间可以有不同的状态。
2. 每个函数对象都有一个自己的类型。 普通的函数只有在函数签名不同时才会有不同的类型。这对于用模板的泛型编程是一个很大的提升，因为你可以将函数行为作为模板参数传递。这样做可以使不同类型的容器利用同一类型的函数对象作为排序标准，确保你不会对不同排序标准的集合进行赋值，组合或者比较。
3. 函数对象通常比普通函数要快。 模板的概念通常允许更好的优化，因为更多细节在编译时定义。因此传递函数对象而不是普通函数通常会带来更好的性能。
示例：
```c++
#include <list>
#include <algorithm>
#include <iostream>
using namespace std;
//function object that adds the value with which it is initialized
class AddValue{
private:
  int theValue;      // the value to add
public:
  // constructor initializes the value to add
  AddValue(int v):theValue(v){}
  //the "function call" for the element adds the value
  void operator()(int& elem) const{
    elem += theValue;
  }
}
```
## 参考文献
Josuttis N M . C++ Standard Library, The: A Tutorial and Reference, 2nd Edition[M]. 2012.
