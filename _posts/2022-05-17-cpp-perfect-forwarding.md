---
title: "완벽한 전달자"
date: 2022-05-17 18:07:53 +9:00
last_modified_at: 2022-05-17 18:07:54 +9:00
categories: cpp
---
완변한 전달자: perfect forwarding
```cpp
#include <iostream>

void foo(int a) {
    std::cout << "foo" << std::endl;
}
void goo(int& a) {
    std::cout << "goo" << std::endl;
    a = 20;
}

template<typename F, typename T>
void HowLong(F f, const T& a) {
    f(a);
}

int main() {
    int n = 0;

    foo(0);
    goo(n);
    
    // 아래 2가지 경우를 모두 완벽하게 전달하는 HowLong 을 만들수 없을까?
    HowLong(foo, 0);    // 상수도 전달하고싶고
    HowLong(goo, n);    // 참조로도 전달하고 싶다.
    std::cout << n << std::endl;
}
```
---
해결책 1. 함수 오버로딩
 - 인자가 한개인 함수의 완벽한 전달자: 2개의 HowLong
 - 인자가 10개인 함수의 완벽한 전달자: 1024개의 HowLong
```cpp
#include <iostream>

void foo(int a) {
    std::cout << "foo" << std::endl;
}
void goo(int& a) {
    std::cout << "goo" << std::endl;
    a = 20;
}

template<typename F, typename T>
void HowLong(F f, const T& a) {
    f(a);
}

template<typename F, typename T>
void HowLong(F f, T& a) {
    f(a);
}

int main() {
    int n = 0;
    HowLong(foo, 0);                // foo
    HowLong(goo, n);                // goo
    std::cout << n << std::endl;    // 20
}
```
이동가능한 참조 만들기: 어려운 내용이다.
 - c++ 참조
   - 값이 이동하는 참조
   - 처음 초기화 된 메모리를 계속 참조한다.
   - 다른 변수를 참조하게 바꿀 수 없다.
 - 포인터
   - 값이 아닌 주소의 이동
   - 가르키는 메모리가 변경 된다
 - 참조는 결국 역참조(*) 되는 포인터 이다.
이동 가능한 참조를 만들어 보자
```cpp
#include <iostream>

template<typename T>
class my_reference_wrapper {
    T* obj; // 메모리를 가르켜야 하므로 결국 포인터 이다.
public:
    my_reference_wrapper(T& r) : obj(&r) {}

    // 진짜 참조와으이 호환을 위해 변환 연산자 제공
    operator T& () {
        return *obj;
    }
};

int main() {
    int n1 = 10;
    int n2 = 20;

    // 진짜 참조로 선언하면 대입시 값 이동이 일어 난다.
    // int &r1 = n1;
    // int& r2 = n2;

    my_reference_wrapper<int> r1 = n1;
    my_reference_wrapper<int> r2 = n2;

    r2 = r1;        // 참조 이동

    int& r3 = r2;   // 진짜 참조와 호환

    std::cout << r3 << std::endl;   // 10
                                    // 값 이동 시   참조 이동 시
    std::cout << n1 << std::endl;   // 10           10
    std::cout << n2 << std::endl;   // 10           20
    std::cout << r1 << std::endl;   // 10           10
    std::cout << r2 << std::endl;   // 10           10
}
```
---
```cpp
#include <iostream>

template<typename T>
class my_reference_wrapper {
    T* obj;
public:
    my_reference_wrapper(T& r) : obj(&r) {}
    operator T& () {
        return *obj;
    }
};

void foo(int a) {
    std::cout << "foo" << std::endl;
}
void goo(int& a) {
    std::cout << "goo" << std::endl;
    a = 20;
}

template<typename F, typename T>
void HowLong(F f, T a) {
    f(a);
}

int main() {
    int n = 10;

    HowLong(foo, 0);                // foo
    // HowLong(goo, n);

    // n 을 참조로 보내고 싶다면 이동 가능한 참조를 사용한다.
    my_reference_wrapper<int> r1 = n;
    HowLong(goo, r1);               // goo
    std::cout << n << std::endl;    // 20
}
```
---
```cpp
#include <iostream>

template<typename T>
class my_reference_wrapper {
    T* obj;
public:
    my_reference_wrapper(T& r) : obj(&r) {}
    operator T& () {
        return *obj;
    }
};

// 클래스 템플릿은 암시적 인자 추론이 불가능해서 항상 어려워 보인다.
// 암시적 인자 추론이 가능한 함수 템플릿으로 Helper 를 만들면 쉬워진다.
template<typename T>
my_reference_wrapper<T> xref(T& a) {
    return my_reference_wrapper<T>(a);
}

void foo(int a) {
    std::cout << "foo" << std::endl;
}
void goo(int& a) {
    std::cout << "goo" << std::endl;
    a = 20;
}

template<typename F, typename T>
void HowLong(F f, T a) {
    f(a);
}

int main() {
    int n = 10;

    // 임시 객체로 만들어서 전달
    HowLong(goo, my_reference_wrapper<int>(n)); // goo

    // 결국 아래처럼 사용
    HowLong(goo, xref(n));          // goo

    std::cout << n << std::endl;    // 20
}
```
---
```cpp
#include <iostream>
#include <functional>
#include <type_traits>              // 여기에 xref 가 이미 정의되어 있다.

void foo(int a, int& b) {
    std::cout << "foo" << std::endl;
    b = 10;
}

int main() {
    int x = 0, y = 0;
    std::function<void()> f;
    f = std::bind(&foo, x, std::ref(y));
    f();                            // foo(x, y);

    std::cout << x << std::endl;    // 0
    std::cout << y << std::endl;    // 10
}
```
---
c++11 의 universal reference 를 사용한 해결책
 - int&&
   - rvalue reference
 - T&&
   - rvalue reference 가 아니다.
   - universal reference 혹은
   - forawrd reference 라고 한다.
```cpp
#include <iostream>

void foo(int a) {
    std::cout << "foo" << std::endl;
}
void goo(int& a) {
    std::cout << "goo" << std::endl;
    a = 20;
}

// universal reference
template<typename F, typename T>
void HowLong(F f, T&& a) {
    f(std::forward<T&&>(a));
}

int main() {
    int n = 0;
    HowLong(foo, 0);                // foo
    HowLong(goo, n);                // goo
    std::cout << n << std::endl;    // 20
}
```
---
```cpp
#include <iostream>

void foo(int& a) {
    std::cout << "int&" << std::endl;
}
void foo(int&& a) {
    std::cout << "int&&" << std::endl;
}

template<typename T>
void Call(T&& a) {
    // T&& 가 int&& 로 결정 되어도 a 자체는 lvalue 이다.
    // 즉, 이름을 가지는 rvalue reference 는 lvalue 이다.

    // 아래 의미처럼 해야 한다.
    // if (is_rvalue_reference<T&&>::value) {
    //     foo(static_cast<T&&>(a));
    // }
    foo(std::forward<T&&>(a));  // std::forward() 가 위의 if 문 처럼 되어 있다.
}

int main() {
    int n = 10;
    Call(n);    // int&
    Call(10);   // int&&
}
```