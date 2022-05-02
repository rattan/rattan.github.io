---
title: "this call 2"
date: 2022-05-02 19:42:04
categories: c++
---
this 관리의 어려움
```cpp
#include <iostream>
#include <windows.h>
#include <conio.h>

DWORD __stdcall foo(void* p) {
    std::cout<<"foo"<<std::endl;
    return 0;
}

int main() {
    CreateThread(0, 0, foo, "A", 0, 0); // 스레드 생성

    _getch();
}
```
 C 의 스레드를 C++ 로 캡슐화 해 보자
 ```cpp
#include <iostream>
#include <windows.h>
#include <conio.h>

class Thread {                                          // 클래스 라이브러리 내부 클래스라고 가정
public:
    void Create() {
        CreateThread(0, 0, _threadMain, this, 0, 0);    // 스레드 함수
                                                        // 1. C 의 callback 함수는 객체지향으로 디자인 될때
                                                        // static 멤버함수가 되어야 한다.
                                                        // 2. static 멤버에는 this 가 없으므로
                                                        // 가상함수나 멤버 data에 접근할 수 없다.
                                                        // 다양한 기법으로 this 를 사용할수 있게 
                                                        // 하는것이 편리하다.
    }

    static DWORD __stdcall _threadMain(void *p) {
        Thread *self = static_cast<Thread*>(p);         // p 가 this 이므로 캐스팅하여 사용

        self->threadMain();                             // 결국 threadMain(self);
        return 0;
    }

    virtual void threadMain() {}                        // threadMain(Thread *this);
};

class MyThread: public Thread {                         // 라이브러리 사용자 클래스
public:
    virtual void threadMain() {
        std::cout<<"My Thread"<<std::endl;
    }
};

int main() {
    MyThread t;
    t.Create();                                         // 이 순간 스레드가 생성되어
                                                        // threadMain 을 수행해야 한다.
    _getch();
}
```