---
title: 【c++ concurrency】 线程管理
date: 2019-03-17 19:44:20
tags: [C++11]
categories: C++
---
## 线程管理基础
### 启动线程
通过构造一个`std::thread`对象来开始一个线程，该对象声明运行在线程上的任务，最简单的是一个无参数返回值为空的普通函数，也可以是带有其他参数的函数对象，总之，`std::thread`对于任何可调用的类型都是有效的，例如，你可以传递一个带有函数调用操作符的类的实例给`std::thread`对象。但对于函数对象，如果传递的是一个临时的对象，线程对象的声明就与函数声明一样了，这时就需要用另外的括号或者用标准的初始化语法。例如：
```
class backgroud_task
{
public:
  void operator()() const
  {
    do_something();
    do_something_else();
  }
};
backgroud_task f;
std::thread my_thread(f);
std::thread my_thread1((backgroud_task())); // right
std::thread my_thread2{backgroud_task()};  // right
```
lambda表达式可以避免这种情况，上边的例子可以写为：
```
std::thread my_thread([]{do_something();
  do_something_else();});
```
### 等待线程结束
如果希望等待一个线程完成，可以在相关的线程实例上调用`join()`函数，可以确保线程在函数退出，局部变量销毁之前线程结束。`jion()`是简单暴力的，要么等待线程结束要么不。`join()`会清理任何与线程相管理的储存，一旦调用`join`，`std::thread`对象就不再是可加入的了，`joinable()`将返回false。
你需要确保在`std::thread`对象销毁之前调用`join()`或者`detach()`，如果你想等待线程，就需要选择合适的地方调用`join()`。例如：
```
struct func
{
  int& i;
  func(int& i_):i(i_){}
  void operator()()
  {
    for(unsigned j =0; j < 1000;++j)
      do_something(i);
  }
}
void f()
{
  int some_local_state=0;
  func my_func(some_local_state);
  std::thread t(my_func);
  try
  {
    do_something_in_current_thread();
  }
  catch(...)
  {
    t.join();
    throw;
  }
  t.join();
}
```
`try/catch`块确保获取局部状态的线程在函数有效之前结束，另外一种方法是使用RAII规则，提供一个类在其析构函数中调用`join()`，例如：
```
class thread_guard
{
  std::thread& t;
public:
  explicit thread_guard(std::thread& t_):
    t(t_){}
  ~thread_guard()
  {
    if(t.joinable())
      t.join();
  }
  thread_guard(thread_guard const&)=delete;
  thread_guard& operator=(thread_guard const&)=delete;
};
struct func;
void f()
{
  int some_local_state=0;
  func my_func(some_local_state);
  std::thread t(my_func);
  thread_guard g(t);
  do_something_in_current_thread();
}
```
### 在后台运行线程
在`std::thread`对象上调用`detach()`使线程在后台运行，这时没有直接的方法与其通信。等待线程结束不再是可能的，也不能获取线程对象的引用，也就不再可加入，所有权和控制转交给c++运行库 。在UNIX概念中称为守护线程(daemon thread)，运行在后台没有任何显示的用户接口。为了从一个`std::thread`对象分离线程，就必须有一个执行线程分离，只有`std::thread`对象`t`当`t.joinable()`返回`true`的时候，才可以调用`t.detach()`。
对于文字处理程序，当处理多个文档窗口时，可以使每个文档编辑窗口运行在其自己的线程中，线程管理请求并不关心等待其他线程结束，因为它在运行在一个不相关的文档上。例如：
```
void edit_document(std::string const& filename)
{
  open_document_and_display_gui(filename);
  while(!done_editing())
  {
    user_command cmd = get_user_input();
    if(cmd.type == open_new_document)
    {
      std::string const new_name = get_filename_from_use();
      std::thread t(edit_document,new_name);
      t.detach();
    }
    else
    {
      process_user_input(cmd);
    }
  }
}
```
## 向线程函数中传递参数
向`std::thread`构造函数传递额外的参数与向函数传参一样简单，要注意的是参数默认按拷贝方式复制到内部存储，执行的创建线程可以从这里获取参数。例如：
```
void f(int i,std::string const& s);
std::thread t(f,3,"hello");
```
当参数是指针的时候需要特别注意，例如：
```
void f(int i,std::string const& s);
void oops(int some_param)
{
  char buffer[1024];
  sprintf(buffer,"%i",some_param);
  std::thread t(f,3,buffer);
  t.detach();
}
```
这里，指向局部变量`buffer`的指针转递给一个新的线程，缓存在新线程中被转换为`std::string`之前，函数`oops`很有可能就退出了，将会导致未定义的行为，解决方法是在传递给线程构造函数前转换成`std::string`类型。
`std::thread t(f,3,std::string(buffer));`
另外一种情形是：对象被复制，你需要的则是引用，当一个线程更新按引用传递的数据结构时会出现这种情况，例如：
```
void update_data_for_widget(widget_id w,widget_data& data);
void oops_again(widget_id w)
{
  widget_data data;
  std::thread t(update_data_for_widget,w,data);
  display_status();
  t.join();
  process_widget_data(data);
}
```
尽管`update_data_for_widget`函数期望第二个参数按引用传递，`std::thread`构造函数却不知道，最后传递给`update_data_for_widget`函数的时data的内部拷贝，而不是data本身的引用，因此传递给`process_widget_data`的时一个没有改变的data。解决方法是使用`std::ref`。
`std::thread t(update_data_for_widget,w,std::ref(data));`
`std::thread`构造函数与`std::bind`工作机制相似，你可以传递一个成员函数指针作为函数，提供合适的对象指针作为第一个参数，例如：
```
class X
{
public:
 void do_lengthy_work();
};
X my_x;
std::thread t(&X::do_lengthy_work,&my_x);
```
另外一种场景是参数不能被复制但是可以被移动(move)，这种类型的一个例子是`std::unique_ptr`，独占式指针，一个`std::unique_ptr`实例只能指向一个给定的对象一次，当实例销毁，指向的对象也被删除，可以利用`std::move`来转移一个动态对象的所有权到线程中。例如：
```
void process_big_object(std::unique_ptr<big_object>);
std::unique_ptr<big_object> p(new big_object);
p->prepare_data(42);
std::thread t(process_big_object,std::move(p));
```
##  转移线程所有权
尽管`std::thread`实例不像`std::unique_ptr`那样拥有动态对象，但它确实拥有资源：每个实例负责管理线程的执行，所有权可以在实例间进行转移，因为`std::thread`实例是`movable`，尽管它们不是`copyable`。这就确保了在任一时刻一个对象与特定的执行线程相关联，并允许开发人员在对象间转移所有权。
```
void some_function();
void some_other_function();
std::thread t1(some_function);
std::thread t2=std::move(t1);
t1 = std::thread(some_other_function);
std::thread t3;
t3 = std::move(t2);
t1 = std::move(t3);
```
&emsp;首先新线程t1创建，其所有权通过`std::move`转移到线程t2，这时t1不再有相关联的线程，运行`some_function`的线程为t2。然后一个新线程启动关联到一个临时`std::thread`对象，随后其所有权转移给t1，这并不需要调用`std::move()`，因为因为所有者是一个临时对象，`move`是自动隐式执行的。t3是默认构造的，并没有关联任何执行的线程，关联到t2的线程的所有权通过`std::move()`转移到t3，这是t1执行的是`some_other_function`，t2没有关联的线程，t3执行的是`some_function`。   
&emsp;最后运行`some_function`的线程的所有权转移回t1，但是这时t1已经关联了一个线程，所以`std::terminate()`被调用终止了程序。在线程析构之前，你必须显式地等待或者分离线程，不能通过向管理它的`std::thread`对象赋新值来丢弃一个线。   
&emsp;如果线程所有权要转移到一个函数中，函数只能以传值方式接受`std::thread`的实例作为参数。例如：
~~~
void f(std::thread t);
void g()
{
  void some_function();
  f(std::threads(some_function));
  std::thread t(some_function);
  f(std::move(t));
}
~~~
`std::thread`支持`move`操作可以使`thread_guard`类实际上获取线程所有权，这避免了`thread_guard`对象比它指向的线程生存时间长的问题，意味着一旦所有权转移到对象中，线程就不再可以被`join`或者`detach`。例如：
~~~
class scoped_thread
{
  std::thread t;
public:
  explicit scoped_thread(std::thread t_):
    t(std::move(t_))
  {
    if(!t.joinable())
      throw std::logic_error("No thread");
  }
  ~scoped_thread()
  {
    t.join();
  }
  scoped_thread(scoped_thread const&)=delete;
  scoped_thread& operator=(scoped_thread const&)=delete;
};
struct func;
void f()
{
  int some_local_state;
  scoped_thread t(std::thread(func(some_local_state)));
  do_something_in_current_thread();
}
~~~
`std::thread`支持`move`操作允许用容器存储线程对象。例如：
~~~
void do_work(unsigned id);
void if()
{
  std::vector<std::thread> threads;
  for(unsigned i = 0; i < 20; ++i)
  {
    threads.push_back(std::thread(do_work,i));
  }
  std::for_each(threads.begin(),threads.end(),std::mem_fn(&std::thread::join));
}
~~~
## 选择线程个数及确认线程
`std::thread::hardware_concurrency()`返回执行程序真正并发运行时的线程个数。
通过`std::thread::id`类型来确认线程，线程对象调用`get_id()`成员函数来获取线程id，当前线程id可以通过调用`std::this_thread::get_id()`来获取。`std::thread::id`类型可以自由地复制和比较。
