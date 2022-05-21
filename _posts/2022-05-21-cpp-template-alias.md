---
title: "template alias"
date: 2022-05-21 19:19:55 +9:00
last_modified_at: 2022-05-21 19:19:56 +9:00
categoris: cpp
---
복잡한 타입은 typedef 먼저 만드는것이 좋다
```cpp
#include <vector>

typedef void(*PF)();

template<typename T>
class MyAlloc {};

typedef std::vector<int, MyAlloc<int>> MyVector;
// 아래 코드처럼 사용하고 싶을때가 많다.
template<typename T>
typedef std::vector<T, MyAlloc<T>> MyVector2;
```
c++ 의 using 은 template 이 된다.
 - template alias
 - template typedef
```cpp
#include <vector>

template<typename T>
class MyAlloc {};

template<typename T>
using MyVector = std::vector<T, MyAlloc<T>>;

// 함수포인터 등의 타입을 define 할때도 사용할 수 있다.
using PF = void(*)();

int main() {
    void(*f)();
    PF f2;
    MyVector<int> v;
}
```
---
```cpp
#include<type_traits>

// const 를 제거한 타입을 구하는 traits
template<typename T>
struct xremove_const {
    typedef T type;
};

template<typename T>
struct xremove_const<const T> {
    typedef T type;
};
// 위처럼 traits 를 만들어 놓고 using 으로 사용하기 쉽게 해주면 된다.

template<typename T>
using xremove_const_t = typename xremove_const<T>::type;

template<typename T>
void foo(T& a) {
    // T 에서 const 를 제거한 타입을 구하는 방법
    typename xremove_const<T>::type n;  // c++11 스타일
    xremove_const_t<T> n2;              // 위와 동일한 표현: c++14 스타일
}

int main() {
    const int c = 0;
    foo(c);
}
```