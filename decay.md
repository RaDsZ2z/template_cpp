# 1.作用与用法
```cpp
//定义于 <type_traits>
template< class T >
struct is_function;
```
1.如果 T 是“U 的数组”或“到 U 的数组的引用”类型，那么成员 typedef type 是 U*。

2.否则，如果 T 是函数类型 F 或到它的引用，那么成员 typedef type 是std::add_pointer<F>::type。

3.否则，成员 typedef type 是 std::remove_cv<std::remove_reference<T>::type>::type。

如果程序添加了 std::decay 的特化，那么行为未定义。

示例
```cpp
//to do
```
# 2.可能的实现
```cpp
template<class T>
struct decay
{
private:
    //去除引用
    typedef typename std::remove_reference<T>::type U;
public:
    typedef typename std::conditional< 
        std::is_array<U>::value,
        //数组则返回元素类型的指针
        typename std::remove_extent<U>::type*,
        typename std::conditional< 
            std::is_function<U>::value,
            //函数则返回函数指针
            typename std::add_pointer<U>::type,
            //不是数组也不是函数则去除cv限定
            typename std::remove_cv<U>::type
        >::type
    >::type type;
};
```
# 3.关于“2.可能的实现”中出现的模板
## 3.1. conditional
```cpp
template<bool B, class T, class F>
struct conditional
```
提供成员 typedef type ，若 B 在编译时为 true 则定义为 T ，或若 B 为 false 则定义为 F 。

如果程序添加了 std::conditional 的特化，那么行为未定义。
```cpp
#include <iostream>    // std::cout
#include <type_traits> // std::is_same

int main()
{
    using T1 = std::conditional<true, int, double>::type;
    using T2 = std::conditional<false, int, double>::type;
    std::cout << std::is_same_v<T1, int> << '\n';
    std::cout << std::is_same_v<T2, double> << '\n';
}
```

## 3.2.is_function
```cpp
template< class T >
struct is_function;
```
检查T是否函数类

std::function、lambda、重载operator()的类和指向函数的指针不是函数类型
```cpp
#include <iostream>    // std::cout
#include <type_traits> // std::is_same
#include <thread>
void f(int x)
{
    std::cout << "x:" << x << '\n';
}
struct X
{
    void operator()()
    {
    }
};
int main()
{
    using T1 = decltype(f);              // 函数类型
    using T2 = std::function<void(int)>; // std::function
    std::function<void(int)> func = f;
    using T3 = decltype(func); // std::cuntion
    auto ff = [](int) -> void {
    };
    using T4 = decltype(ff);  // lambda
    using T5 = X;             // 重载()的类
    using T6 = void (*)(int); // 函数指针
    std::cout << std::boolalpha;
    std::cout << std::is_same_v<T2, T3> << '\n'; // true  T2 == T3
    std::cout << "-----\n";
    std::cout << std::is_function_v<T1> << '\n'; // true
    std::cout << std::is_function_v<T2> << '\n'; // false
    std::cout << std::is_function_v<T3> << '\n'; // false
    std::cout << std::is_function_v<T4> << '\n'; // false
    std::cout << std::is_function_v<T5> << '\n'; // false
    std::cout << std::is_function_v<T6> << '\n'; // false
}
```
## 3.3.remove_extent
单独写了一个.md文件

# 4.数组引用与函数引用
## 4.1.数组引用与函数引用
```cpp
#include <iostream>    // std::cout
#include <type_traits> // std::is_same
void f(int, int)
{
}
int main()
{
    using T1 = int(&)[];  //  int[]类型的引用
    using T2 = int(&)[5]; // int[5]类型的引用

    using T3 = void (&)(int, int); // void (int,int)函数类型 的引用
    
    std::cout << std::is_same_v<decltype(f), std::remove_reference_t<T3>>;
}
```
## 4.2.函数指针
既然聊到了函数引用，你可能也联想到了函数指针，我们来写一下
```cpp
#include <iostream>    // std::cout
#include <type_traits> // std::is_same
void f(int, int)
{
}
int main()
{
    using T = void (*)(int, int); // void (int,int)函数类型 的指针
    std::cout << std::is_same_v<decltype(f), std::remove_pointer_t<T>> << '\n';

    std::cout << std::is_same_v<std::add_pointer_t<decltype(f)>, T> << '\n';
}
```
关于函数指针我又发现了一些 ~~有趣~~ 抽象的东西，放一段代码不解释了
```cpp
#include <iostream>    // std::cout
#include <type_traits> // std::is_same
#include <thread>
#include <stdio.h>
void f(int x) { std::cout << "hello world!\n"; }

void f1(void (*f)(int), int y)
{
    f(y);
}

void f2(int x)
{
    std::cout << "x:" << x;
}
int main()
{
    void (*p)(int) = f;
    std::thread t1{*****p, 3};
    t1.join();

    std::thread t2{*****f, 3};
    t2.join();

    std::thread t3{********************************f, 3};
    t3.join();

    f1(********************f2, 3);
}

```
## 4.3.数组指针
那有没有数组指针这种东西呢？当然也是有的
```cpp
#include <iostream>    // std::cout
#include <type_traits> // std::is_same
int main()
{
    using T1 = int(*)[]; //数组指针
    using T2 = int[];    //数组
    using T3 = T2 *;     //数组指针
    std::cout << std::boolalpha;
    std::cout << std::is_same_v<T1, T3> << '\n';                     // true
    std::cout << std::is_same_v<T1, std::add_pointer_t<T2>> << '\n'; // true
}
```
参考 [cpperference](https://zh.cppreference.com/w/cpp/types/decay)
