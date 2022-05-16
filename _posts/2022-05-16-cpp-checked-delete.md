---
title: "checked delete"
date: 
last_modified_at: 
categories: cpp
---
Checked delete
 - boost 팀이 최초로 만든 개념
```cpp
#include <iostream>

class Test; // 클래스 전방 선언
            // 완전한 선언이 없어도 포인터 변수를 사용할 수 있다.

Test* p;    // incomplete object(불완전한 객제)
            // 전방선언만 있는 타입의 포인터
            // delete 하면 소멸자를 호출하지 못한다.

// checked delete
void foo(Test* pt) {
    enum { type_must_be_complete = sizeof(Test) };  // 불완전한 객체의 크기는 구할 수 없다.
    delete pt;
}

class Test {
public:
    Test() {
        std::cout << "생성자" << std::endl;
    }
    ~Test() {
        std::cout << "소멸자" << std::endl;
    }
};

int main() {
    Test* p = new Test;
    foo(p);
}
```