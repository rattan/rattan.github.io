---
title: "완벽한 전달자2"
date: 2022-05-21 20:40:10 +9:00
last_modified_at: 2022-05-21 20:40:11 +9:00
categories: cpp
---
함수 인자의 개수는 다양하다.
 - 가변인자 템플릿으로 만들자
```cpp
#include <iostream>

int g_x = 0;

int& foo(int a, int& b) {
    std::cout<<"foo"<<std::endl;
    b = 10;
    return g_x;
}

template<typename F, typename ...Types>
void HowLong(F f, Types ...args) {
    f(args...);
}

int main() {
    int x = 0;
    HowLong(foo, 1, x); // foo
}
```
---
참조인자를 가지는 함수도 생각해야 한다.
 - 가변인자를 universal reference 로 받아야 한다.
```cpp
#include <iostream>

int g_x = 0;

int& foo(int a, int& b) {
    std::cout<<"foo"<<std::endl;
    b = 10;
    return g_x;
}

template<typename F, typename ...Types>
void HowLong(F f, Types&& ...args) {
    f(args...);
}

int main() {
    int x = 0;
    HowLong(foo, 1, x);     // foo
    std::cout<<x<<std::endl;// 10
}
```
---
리턴 값을 돌려주어야 한다.
```cpp
#include <iostream>

int g_x = 0;

int& foo(int a, int& b) {
    std::cout<<"foo"<<std::endl;
    b = 10;
    return g_x;
}

template<typename F, typename ...Types>
decltype(auto) HowLong(F f, Types&& ...args) {
    return f(args...);
}

int main() {
    int x = 0;
    auto& r = HowLong(foo, 1, x);   // foo
    r = 30;
    std::cout<<g_x<<std::endl;      // 30
}
```
rvalue 가 가변인자로 전달되면 Types&& 은 rvalue reference 이다. 하지만 args... 자체는 이름있는 rvalue reference 이므로 lvalue 가 된다.
 - forward<> 로 묶어야 한다.
```cpp
#include <iostream>

void foo(int&& a, int&& b) {
    std::cout<<"foo"<<std::endl;
}

template<typename F, typename ...Types>
decltype(auto) HowLong(F f, Types&& ...args) {
    return f(std::forward<Types&&>(args)...);   // f(std::forward<T&&>(0), std::forward<T&&>(0));
}

int main() {
    foo(0, 0);          // foo
    HowLong(foo, 0, 0); // foo
}
```
---
```cpp
#include <iostream>
#include <chrono>

using TP = std::chrono::steady_clock::time_point;

class Timer {
    TP start;
public:
    Timer() {
        start = std::chrono::high_resolution_clock::now();
    }
    ~Timer() {
        auto end = std::chrono::high_resolution_clock::now();
        std::cout<<"걸린 시간: "<<std::chrono::duration_cast<std::chrono::milliseconds>(end - start).count()<<std::endl;
    }
};

void foo() {
    std::cout<<"foo"<<std::endl;
    for(int i = 0;i<10000000;++i);
}

template<typename F, typename ...Types>
decltype(auto) HowLong(F f, Types&& ...args) {
    Timer tm;
    return f(std::forward<Types&&>(args)...);
}

int main() {
    HowLong(foo);   // foo
                    // 걸린 시간: 28
}
```
