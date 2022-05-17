---
title: "타입 추론"
date: 2022-05-17 18:54:43 +9:00
last_modified_at: 2022-05-17 18:54:44 +9:00
categories: cpp
---
타입 추론: Type Deduction<p>
규칙1.
 - ParamType 이 포인터나 참조가 아닐때, expr 의 const, volatile, reference 속성을 제거하고 전달된다.
```cpp
template<typename T>
void goo(const T& a) {
    // 아래 표현에서 a 의 타입을 ParamType 이라고 부른다.
    // T: int a: const int&
}

template<typename T>
void foo(T a) {}

int main() {
    int n = 0;
    const int c = 0;
    const int& r = c;
    foo(n); // T: int
    foo(c); // T: int
    foo(r); // T: int
}
```
---
규칙2.
 - ParamType 이 포인터나 참조일때
   - expr 이 레퍼런스 라면 레퍼런스만 무시된다.
   - const 는 유지된다.
   - expr 을 고려해서 타입을 결정한다.
```cpp
template<typename T>
void foo(T& a) {}

template<typename T>
void goo(const T& a) {}

int main() {
    int n = 10;
    const int c = n;
    const int& r = c;

    foo(n); // T: int       a: int&
    foo(c); // T: const int a: const int&
    foo(r); // T: const int a: const int&

    goo(n); // T: int       a: const int&
    goo(c); // T: int       a: const int&
    goo(r); // T: int       a: const int&
}
```
---
```cpp
#include <iostream>

int x = 10;

int& foo() {
    return x;
}

int main() {
    auto n = foo(); // 규칙1. 우변의 타입 중 const, reference, volatile 을 버린다.
                    // n 의 타입은?: tempalte 과 동일
                    // auto: T  n: param    expr: = foo()

    auto& r = foo();    // 규칙2. 우변의 속성중 레퍼런스 무시
                        // auto: int    r: int&

    r = 20;
    std::cout << x << std::endl;    // 20
}
```
---
규칙3.
 - universal reference 인 경우
   - int&&: rvalue reference: rvalue 만 담을 수 있다.
   - T&&: universal reference/forward reference
```cpp
// 인자로 전달되는 표현이 lvalue 이면, T&& 는 lvalue reference 가 되고
// 인자로 전달되는 표현이 rvalue 이면, T&& 는 rvalue reference 가 된다.
template<typename T>
void foo(T&& a) {}

int main() {
    int n = 10;
    foo(n);     // lvalue reference
    foo(10);    // rvalue reference

    int& r = n;
    foo(r);     // lvalue reference
}
```
  
