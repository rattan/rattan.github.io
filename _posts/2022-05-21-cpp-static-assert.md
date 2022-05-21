---
title: "static assert"
date: 2022-05-21 19:06:49 +9:00
last_modified_at: 2022-05-21 19:06:50 +9:00
categories: cpp
---
인자, 리턴값 등은 무조건 사용하지 말고 값이 유효한지 확인하는것이 좋다.
 - assert

```cpp
#include <assert.h>

// 기존의 assert(): 실행시간 동작하는 함수
void foo(int age) {
    assert(age > 0);    // 조건을 만족하지 않으면 abort() 를 호출하고 종료
    
    char* p = new char[age];
}

int main() {
    foo(-10);   // Assertion failed
}
```
---
```cpp
#include <iostream>
#include <type_traits>

static_assert(sizeof(double) >= 8, "double 이 8 보다 커야 컴파일 된다.");

#pragma pack(1)
struct Packet {
    char cmd;
    int data;
};
// Packet 의 크기는?
// 구조체에 padding 이 있다면, 컴파일을 멈추게 해 보자
satatic_assert(sizeof(Packet) == sizeof(char) + sizeof(int), "unexpected padding");

class Test {
    int data;
public:
    virtual void foo() {}
};

template<typename T>
void foo(T a) {
    static_assert(!std::is_polymorphic<T>::value, "T has vertual function");
    
    T tmp;
    memset(&tmp, 0, sizeof(T));
}

int main() {
    Test t;
    foo(t);
}
```