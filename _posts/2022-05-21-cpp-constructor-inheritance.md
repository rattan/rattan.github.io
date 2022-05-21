---
title: "생성자 상속"
date: 
last_modified_at: 
categories: cpp
---
```cpp
#include <iostream>

class Base {
public:
    void foo(int) {
        std::cout<<"int"<<std::endl;
    }
    void foo(int, int) {
        std::cout<<"int, int"<<std::endl;
    }
};

class Derived: public Base {
public:
    // 자식에 의해 가려진 부모의 함수를 사용하고 싶다
    using Base::foo;
    
    // Hidden
    // 자식이 foo 라는 이름을 사용하면 부모에 있는 모든 foo 함수는 가려져 사용할 수 없다.
    void foo(double) {
        std::cout<<"double"<<std::endl;
    }
};

int main() {
    Derived d;
    
    d.foo(0);       // int
    d.foo(1, 2);    // int, int
    d.foo(3.4);     // double
}
```
---
c++11 에서는 using 을 생성자도 사용 가능하게 되었다.
```cpp
#include <iostream>

class Base {
    int data;
public:
    Base(): data(0) {}
    Base(int a): data(a) {}
};

class Derived: public Base {
public:
    // 부모의 생성자도 상속해 달라
    using Base::Base;
    Derived() {}
};

int main() {
    Derived d1;
    Derived d2(1);
}
```