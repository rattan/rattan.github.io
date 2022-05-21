---
title: "이름 충돌"
date: 2022-05-21 20:44:56 +9:00
categories: cpp
---
```cpp
#include <iostream>

class A {
public:
    int value;
    A(): value(10) {}
};
class B {
public:
    int value;
    B():value(20) {}
};

class C: public A, public B {};

int main() {
    C c;
    std::cout<<c.value<<std::endl;      // error.
                                        // A 의 value 인지 B 의 value 인지 결정할 수 없다.
    
    std::cout<<c.A::value<<std::endl;   // 10
                                        // c 의 객체에서 A 클래스의  value 에 접근지정
    std::cout<<((B)c).value<<std::endl; // 20
                                        // c 를 캐스팅 해서 사용
}
```