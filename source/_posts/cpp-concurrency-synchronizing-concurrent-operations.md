---
title: cpp-concurrency-synchronizing-concurrent-operations
date: 2020-09-22 17:03:28
tags: [C++11]
categories: C++
---

## 等待一个事件或者其他条件
有些时候我们不仅是需要保护数据同时需要在线程间同步操作，比如希望一个线程等待一个特定的事件发生或者一个条件为真。如果一个线程正在等待第二个线程完成一个任务，有几种方式，第一，可以一直检测共享数据中的标志位，让第二个线程完成任务后设置标志位，这种方式非常浪费资源。第二个选择是让等待线程利用`std::this_thread::sleep_for()`函数在检查之间睡眠一段时间，这种方式比第一种有所提升，在线程睡眠时不需要浪费处理时间，但是很难设定合适的睡眠周期。第三种方式是使用c++标注库中的`condition variables`。条件变量与某个事件或条件相关联，一个或多个线程可以等待条件满足，当一个线程决定条件变量受到满足，然后就可以通知一个或多个等待条件变量的线程，唤醒它们并允许它们继续处理。

## 使用条件变量等待一个条件
标准库提供了`std::condition_variable`和`std::condition_variable_any`，前者与`std::mutex`配合使用，后者与任何满足最小条件的类似互斥量工作。
```
std::mutex m;
std::queue<data_chunk> data_queue;
std::condition_variable data_cond;

void data_preparation_thread()
{
    while(more_data_to_prepare())
    {
        data_chunk const data = prepare_data();
        std::lock_guard<std::mutex> lk(m);
        data_queue.push(data);
        data_cond.notify_one();
    }
}

void data_processing_thread()
{
    while(true)
    {
        std::unique_lock<std::mutex> lk(m);
        data_cond.wait(lk, []{return !data_queue.empty();});
        data_chunk data=data_queue.front();
        data_queue.pop();
        lk.unlock();

        process(data);
        if(is_last_chunk(data))
            break;
    }
}
```
对于数据准备线程，当数据准备好时，线程利用`std::lock_guard`对互斥量加锁保护队列，并把数据加入到队列中，接着调用`notify_one()`通知等待线程。数据处理线程首先使用`std::unique_lock`对互斥量加锁，然后调动`wait()`传入等待条件的函数对象。`wait()`的实现检查条件是否满足，如果不满足，`wait()`对互斥量解锁，阻塞线程或进入等待状态。当条件通过数据准备线程`notify_one()`被通知时，数据处理线程被唤醒，获取互斥量上的锁，再次检查条件，如果条件满足从`wait()`返回的互斥量依然是加锁的
使用条件变量构建线程安全队列
```
#include <queue>
#include <memory>
#include <mutex>
#include <condition_variable>

template<typename T>
class threadsafe_queue
{
private:
    mutable std::mutex mut; // the mutex must be mutable
    std::queue<T> data_queue;
    std::condition_variable data_cond;
public:
   threadsafe_queue(){}
   threadsafe_queue(threadsafe_queue const& other)
   {
       std::lock_guard<std::mutex> lk(other.mut);
       data_queue = other.data_queue;
   }

   void push(T new_value)
   {
       std::lock_guard<std::mutex> lk(mut);
       data_queue.push(new_value);
       data_cond.notify_one();
   }

   void wait_and_pop(T& value)
   {
       std::unique_lock<std::mutex> lk(mut);
       data_cond.wait(lk, [this]{return !data_queue.empty();});
       value=data_queue.front();
       data_queue.pop();
   }

   std::shared_ptr<T> wait_and_pop()
   {
       std::unique_lock<std::mutex> lk(mut);
       data_cond.wait(lk, [this]{return !data_queue.empty();});
       std::shared_ptr<T> res(std::make_shared<T>(data_queue.front()));
       data_queue.pop();
       return res;
   }

   bool try_pop(T& value)
   {
       std::lock_guard<std::mutex> lk(mut);
       if(data_queue.empty())
           return false;
       value=data_queue.front();
       data_queue.pop();
       return true;
   }

   std::shared_ptr<T> try_pop()
   {
       std::lock_guard<std::mutex> lk(mut);
       if(data_queue.empty())
           return std::shared_ptr<T>();
       std::shared_ptr<T> res(std::make_shared<T>(data_queue.front()));
       data_queue.pop();
       return res;
   }

   bool empty() const
   {
       std::lock_guard<std::mutex> lk(mut);
       return data_queue.empty();
   }
};
```
尽管`empty()`是一个常量成员函数，复制构造函数中的other参数是一个常量引用，其他线程可能有对象的非常量引用，会调用能够修改的成员函数，因此我们依然需要对互斥量加锁，由于这是一个修改操作，互斥量必须标记为`mutable`。
条件变量对于多个线程等待相同事件也是有用的，如果新数据已经准备，调用`notify_one()`会触发某一个线程执行`wait()`检查条件并从`wait()`返回。这并不保证哪个线程被通知；另一种可能是多个线程等待同一个事件，但是所有的线程都需要回应。这时数据准备线程可以调用`notify_all()`,会触发所有当前执行`wait()`的线程检查等待的条件；如果等待线程只等待一次，如果条件是true，线程将不再等待这个条件变量，这时就需要使用future。

## 使用futures等待一次性事件
C++标准库使用futrue建模一次性事件。如果一个线程需要等待一个特定的一次性事件，可以说它获得了代表这个事件的一个期望。
线程可以在一段事件内周期地等待期望看是否事件发生，在检测之间可以执行其他的任务。C++标准库实现了两个future的模板类：
`unique futures(std::future<>)`和`shared futures（std::shared_future<>)`。`std::future`是指向它关联事件的仅有的一个实例。`std::shared_future`的多个实例可以指向同一个事件。

## 从后台任务中返回值
假设你有一个需要长时间运行的计算，并期望最终会得到一个有用的结果，但现在并不需要这个值。你可以开启一个新的线程去执行计算，但是
这意味着你必须要把结果转移回来，`std::thread`并不提供这样的机制。这时就需要`std::async`，使用`std::async`可以开启一个你并不马上需要结果的异步任务，`std::async`返回一个`std::future`对象，会包含函数的返回值。当你需要该值的时候，只需要在future对象上调用`get()`函数。线程就会阻塞直到future对象ready，并返回值。
```
#include <future>
#include <iostream>

int find_the_answer_to_ltuae();
void do_other_stuff();
int main()
{
    std::future<int> the_answer = std::async(find_the_answer_to_ltuae);
    do_other_stuff();
    std::cout<<"The answer is "<<the_answer.get()<<std::endl;
}

与std::thread类似，std::async允许你传递给函数额外的参数，例如：
#include <string>
#include <future>
struct X
{
    void foo(int, std::string const&);
    std::string bar(std::string const&);
};
X x;
auto f1=std::async(&X::foo, &x, 42, "hello"); // calls p->foo(42, "hello") where p is &x
auto f2=std::async(&X::bar, x, "goodbye"); // calls tempx.bar("goodbye") where tempx is a copy of x
struct Y
{
    double operator(double);
};
Y y;
auto f3=std::async(Y(), 3.141); // calls tmpy(3.141) where tmpy is move-constructed from Y()
auto f4=std::async(std::ref(y),2.718); // calls y(2.718)
X baz(X&);
std::async(baz, std::ref(x)); // calls baz(x)
class move_only
{
public:
    move_only();
    move_only(move_only&&)
    move_only(move_only const&) = delete;
    move_only& operator=(move_only&&);
    move_only& operator=(move_only const&) = delete;

    void operator()();
};
auto f5 = std::async(move_only()); // calls tmp() where tmp is constructed from std::move(move_only())
```
`std::async`还可以接受`std::launch`类型的参数，可以指定`std::launch::deferred`或`std::launch::async`。前者指示函数调用将被延迟，直到
在future上调用`wait()`或者`get()`。后者指示函数必须运行在自己的线程之上。还可以指定s`td::launch::deferred|std::launch::async`来表明实现可以选择

```
auto f6=std::async(std::launch::async, Y(), 1.2); // Run in new thread
auto f7=std::async(std::launch::deferred, baz, std::ref(x)); // Run in wait() or get()
auto f8=std::async(std::launch::deferred|std::launch::async, baz, std::ref(x)); // Implementation chooses
auto f9 = std::async(baz, std::ref(x)); // Implementation chooses
f7.wait(); // Invoke deferred function

```
## 任务与期望关联
`std::packaged_task<>`可以将期望绑定到一个函数或可调用对象上。当`std::packaged_task<>`对象调用，它会调用关联的函数或可调用对象，使期望就绪，并将返回值存储为关联数据。`std::packaged_task<>`类模板的模板参数是一个函数签名，从`get_future()`成员函数返回的`std::future<>`的类型由指定函数签名的返回值确定。函数签名的参数列表用来说明任务的函数调用操作符的签名。例如：
```
template<>
class packaged_task<std::string(std::vector<char>*, int)>
{
public:
    template<typename Callable>
    explicit packaged_task(Callable&& f);
    std::future<std::string> get_future();
    void operator()(std::vector<char>*, int);
};
```
`std::packaged_task`对象是一个可调用对象，可以封装到`std::function`对象中，传递给`std::thread`作为线程函数，传递给另一个
需要一个可调用对象的函数，或者直接调用。
许多GUI框架需要特定的线程来完成GUI的更新，所以如果其他线程需要更新GUI，它必须发送一个消息给到正确的线程。std::packaged_task提供
了完成这种功能的一种方法，且不需要发送自定义信息给GUI相关线程。
```
#include <deque>
#include <mutex>
#include <future>
#include <thread>
#include <utility>

std::mutex m;
std::deque<std::packaged_task<void()>> tasks;

bool gui_shutdown_message_received();
void get_and_process_gui_message();

void gui_thread()
{
    while(!gui_shutdown_message_received())
    {
        get_and_process_gui_message();
        std::packaged_task<void()> task;
        {
            std::lock_guard<std::mutex> lk(m);
            if(tasks.empty())
                continue;
            
            task=std::move(tasks.front());
            tasks.pop_front();
        }
        task();
    }
}

std::thread gui_bg_thread(gui_thread);

template<typename Func>
std::future<void> post_task_for_gui_thread(Func f)
{
    std::package_task<void()> task(f);
    std::future<void> res = task.get_future();
    std::lock_guard<std::mutex> lk(m);
    tasks.push_back(std::move(task));
    return res;
}
```

## 使用std::promises

`std::promise<T>`提供一种设置值（类型T)的方式，可以通过一个关联的`std::future<T>`对象来读取。
一个`std::promise/std::future`组合为这种方式提供了一种可行的机制；等待线程在期望上阻塞，而提供数据的
线程可以使用承诺来设置相关值，并使期望就绪。
像`std::packaged_task`一样，你可以通过调用`get_future()`成员函数获得给定`std::promise`关联的`std::future`对象。
当承诺的值设定后，期望变为就绪状态，可以用来获取存储的值。如果你在设置值之前销毁`std::promise`，将会存储一个异常。
下面的例子是一个线程处理多个链接的代码。这个例子中，使用`std::promise<bool>/std::future<bool>`组合来确定输出数据的
成功发送，和期望相关的值只是简单的成功/失败标识。对于输入包，和期望相关的数据是数据包中的有效载荷。
```
#include <future>

void process_connections(connection_set& connections)
{
  while(!done(connections))  // 1
  {
    for(connection_iterator  // 2
            connection=connections.begin(),end=connections.end();
          connection!=end;
          ++connection)
    {
      if(connection->has_incoming_data())  // 3
      {
        data_packet data=connection->incoming();
        std::promise<payload_type>& p=
            connection->get_promise(data.id);  // 4
        p.set_value(data.payload);
      }
      if(connection->has_outgoing_data())  // 5
      {
        outgoing_packet data=
            connection->top_of_outgoing_queue();
        connection->send(data.payload);
        data.promise.set_value(true);  // 6
      }
    }
  }
}
```

## 为期望存储异常

`std::promise`提供了`set_exception()`成员函数来存储异常

```
extern std::promise<double> some_promise;
try
{
    some_promise.set_value(calculate_value());
}
catch(...)
{
    some_promise.set_exception(std::current_exception());
}
```
另外一种方式是使用`std::copy_exception()`
```
some_promise.set_exception(std::copy_exception(std::logic_error("foo ")));
```

## 等待多个线程

尽管`std::future`处理将数据从一个线程到另一个线程的所有同步需求，调用一个特定`std::futrue`实例的成员函数在线程之间是没有同步的。
如果你从多个线程中访问`std::future`对象，没有进行同步，将会遇到data race和未定义行为。标准库提供了`std::shared_future`来处理这种情况，`std::future`只是moveable，`std::shared_future`实例是copyable的。由于std::future对象不和其他对象共享异步状态的所有权，所以需要使用`std::move`将所有权从`std::future`转移给`std::shared_future`，例如：

```
std::promise<int> p;
std::future<int> f(p.get_future());
assert(f.valid()); // the future f is valid
std::shared_future<int> sf(std::move(f)); 
assert(!f.valid()); // f is no longer valid
assert(sf.valid()); // sf is now valid
```

## 等待一段时间

有两种等待一段时间的方式:基于时间段duration-base，绝对的时间点，前者通常有_for形式的后缀，后者以_until结尾。

```
// Waiting for a condition variable with a timeout
#include <condition_variale>
#include <mutex>
#include <chrono>

std::condition_variable cv;
bool done;
std::mutex m;

bool wait_loop()
{
    auto const timeout = std::chrono::steady_clock::now() + std::chrono::milliseconds(500);
    std::unique_lock<std::mutex> lk(m);
    while(!done)
    {
        if(cv.wait_until(lk,timeout) == std::cv_status::timeout)
           break;
    }
    return done;
}
```

使用暂停最简单的例子是在一个线程处理中添加延迟，这样就不会在自己不工作的时候，不会占用其他线程的处理时间，常用的两个函数
是`std::this_thread::sleep_for()`和`std::this_thread::sleep_until()`。

## 使用同步操作来简化代码

函数式风格的QuichSORT
```
// A sequential implementation of quicksort
template<typename T>
std::list<T> sequential_quick_sort(std::list<T> input)
{
    if(input.empty())
    {
        return input;
    }
    std::list<T> result;
    result.splice(result.begin(), input, input.begin());
    T const& pivot = *result.begin();

    auto divide_point=std::partition(input.begin(), input.end(), [&](T const& t){return t<pivot;});
    std::list<T> lower_part;
    lower_part.splice(lower_part.end(),input,input.begin(),divide_point);

    auto new_lower(sequential_quick_sort(std::move(lower_part)));
    auto new_higher(sequential_quick_sort(std::move(input)));

    result.splice(result.end(),new_higher);
    result.splice(result.begin(),new_lower);
    return result;
}
```

函数式风格的并行Quicksort

```
template<typename T>
std::list<T> parallel_quick_sort(std::list<T> input)
{
    if(input.empty())
    {
        return input;
    }
    std::list<T> result;
    result.splice(result.begin(),input, input.begin());
    T const& pivot = *result.begin();
    auto divide_point=std::partition(input.begin(),input.end(),[&](T const& t){return t<pivot;});

    std::list<T> lower_part;
    lower_part.splice(lower_part.end(),input,input.begin(),divide_point);
    std::future<std::list<T>> new_lower(std::async(&parallel_quick_sort<T>,std::move(lower_part)));

    auto new_higher(parallel_quick_sort(std::move(input)));

    result.splice(result.end(), new_higher);
    result.splice(result.begin(), new_lower.get());
    return result;
}
```