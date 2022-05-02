---
published: true
title: "this call"
date: 2022-04-29 20:46:25
last_modified_at: 2022-05-02 21:38:46
categories: cpp
---
1. 멤버함수의 호출 원리
   - 객체가 함수의 1번째 인자(this) 로 추가 된다. - this call
   - 정확히는 exc 레지스터로 전달
2. static 멤버 함수는 this 가 추가되지 않는다.
```cpp
class Point {
    int x, y;
public:
    void set(int a, int b) {    // void set(Point* const this, int a, int b)
        x = a;                  // this->x = a;
        y = b;                  // this->y = b;
    }

    static void foo(int a) {    // void foo(int a)
        x = a;                  // error C2597: 비정적 멤버 'Point::x'에 대한 참조가 잘못되었습니다.
                                // this->x = a 가 되어야 하는데 this 가 없다.
    }                           // 그래서 static 멤버에서는 멤버변수 접근이 안된다.
};

int main() {
    Point::foo(10);             // static 멤버함수는 객체없이 호출 가능
                                // push 10
                                // 보낼 객체가 없다.
                                // call Point::foo

    Point p1, p2;
    p1.set(10, 20);             // set(&p1, 10, 20) 이 된다.
                                // push 20
                                // push 10 매개변수는 스택으로
                                // mov ecx, &p1 객체 주소는 레지스터에
                                // call Point::set
}
```
>error code
>- [C2597 비정적 멤버 참조는 특정 개체에 상대적이어야 합니다.](https://docs.microsoft.com/ko-kr/cpp/error-messages/compiler-errors-2/compiler-error-c2597)
---
1. 일반 함수 포인터에 멤버함수의 주소를 담을 수 없다. this 때문에.
2. 일반 함수 포인터에 static 멤버함수의 주소를 담을 수 있다.
```cpp
#include<iostream>

class Dialog {
public:
    void Close() { std::cout << "Dialog close" << std::endl; }
};

void foo() { std::cout << "foo" << std::endl; }

int main() {
    void(*f1)() = &foo;                     // ok.
    void(*f2)() = &Dialog::Close;           // error C2440: '초기화 중': 'void (__cdecl Dialog::* )(void)'에서 'void (__cdecl *)(void)'(으)로 변환할 수 없습니다.

    void(Dialog:: * f3)() = &Dialog::Close; // ok. 멤버함수 포인터를 만드는 법
    f3();                                   // error C2064: 항은 0개의 인수를 받아들이는 함수로 계산되지 않습니다.
                                            // this 가 없다.

    Dialog dlg;
    dlg.f3();                               // error C2039: 'f3': 'Dialog'의 멤버가 아닙니다.
                                            // dlg.Close() 즉 f3(&dlg)
                                            // 하지만 이 경우 컴파일러가 f3 이라는 멤버를 찾게되어 error

    (dlg.*f3)();                            // f3 은 함수포인터 이므로 *f3 하면 함수가 된다.
                                            // .*연산자 우선 순위를 함수호출() 보다 높여야 한다.

    Dialog* pDlg = &dlg;
    (pDlg->*f3)();                          // 포인터인 경우 arrow operator 로 호출할 수 있다.
}
```
>error code
>- [C2440 "void(Dialog::*)()"형식의 값을 사용하여 "void(*)()"형식의 엔터티를 초기화할 수 없습니다.](https://docs.microsoft.com/ko-kr/cpp/error-messages/compiler-errors-1/compiler-error-c2440)
>- [C2064 명백한 호출의 괄호 앞에 오는 식에는 함수 (포인터) 형식이 있어야 합니다.](https://docs.microsoft.com/ko-kr/cpp/error-messages/compiler-errors-1/compiler-error-c2064)
>- [C2039 클래스 "Dialog"에 "f3" 멤버가 없습니다.](https://docs.microsoft.com/ko-kr/cpp/error-messages/compiler-errors-1/compiler-error-c2039)
---
this 관리의 어려움
```cpp
#include <iostream>
#include <windows.h>
#include <conio.h>

DWORD __stdcall foo(void* p) {
    std::cout << "foo" << std::endl;
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

    static DWORD __stdcall _threadMain(void* p) {
        Thread* self = static_cast<Thread*>(p);         // p 가 this 이므로 캐스팅하여 사용

        self->threadMain();                             // 결국 threadMain(self);
        return 0;
    }

    virtual void threadMain() {}                        // threadMain(Thread *this);
};

class MyThread : public Thread {                         // 라이브러리 사용자 클래스
public:
    virtual void threadMain() {
        std::cout << "My Thread" << std::endl;
    }
};

int main() {
    MyThread t;
    t.Create();                                         // 이 순간 스레드가 생성되어
                                                        // threadMain 을 수행해야 한다.
    _getch();
}
```
---
NULL 객체의 함수 호출 문제
```cpp
#include <iostream>

class Test {
    int data;
public:
    void f1() {
        std::cout << "f1" << std::endl;
    }
    int f2() {
        std::cout << "f2" << std::endl;
        return 0;
    }
    int f3() {
        std::cout << "f3" << std::endl;
        return data;                // this->data;
    }

    int call_f3() {
        return this ? f3() : 0;     // NULL 객체에 대해 함수를 호출 해도 죽지않게 하기 위한 함수
    }

    virtual void f4() {}
};

int main() {
    Test* p = nullptr;              // 메모리 할당에 실패해 nullptr 이 나왔다고 가정

    p->f1();                        // f1(p), f1(nullptr) : 어떻게 될까?
    p->f2();                        // ok.
    p->f3();                        // error. this이(가) nullptr였습니다.
    p->call_f3();                   // ok.
    p->f4();                        // nullptr 번지의 가상함수 테이블을 찾으려 한다.
                                    // runtime error.
}
```
---
다중상속과 this, 그리고 함수 포인터
```cpp
#include <iostream>

class X {
public:
    int x;
    void fx() {
        std::cout << this << std::endl;
    }
};
class Y {
public: int y;
      void fy() {
          std::cout << this << std::endl;
      }
};
class C : public X, public Y {
public: int c;
};

int main() {
    C ccc;
    std::cout << &ccc << std::endl; // 100번지라고 할때,

    X* pX = &ccc;
    Y* pY = &ccc;
    std::cout << pX << std::endl;   // 100번지
    std::cout << pY << std::endl;   // 104번지

    ccc.fx();                       // 100번지
    ccc.fy();                       // 104번지

    void(C:: * f)();                // 16Byte { 함수 주소, this offset }
    f = &C::fx;                     // f = { fx 주소, 0 }
    (ccc.*f)();                     // f(&ccc), 100번지
    f = &C::fy;                     // f = { fy 주소, sizeof(x) 즉 4 }
    (ccc.*f)();                     // f(&ccc), 104번지
                                    // f.함수주소(&ccc + f.this_offset)
    std::cout << sizeof(f) << std::endl;
}
```
---
가상함수의 주소를 꺼내면 진짜 주소가 아닌 가상함수 테이블의 index,
즉 가상함수의 순서가 나오게 된다.
 - g++, xCode : 0, 1, 2, 3 등의 숫자
 - VC++ : 주소비슷하게 나오는데 그 주소를 따라가면 index 가 나온다.
```cpp
#include <iostream>

class Base {
public:
    virtual void goo() {
        std::cout << "Base goo" << std::endl;
    }
    virtual void foo() {
        std::cout << "Base foo" << std::endl;
    }
    virtual void hoo() {
        std::cout << "Base hoo" << std::endl;
    }
};

class Derived : public Base {
public:
    virtual void foo() {
        std::cout << "Derived foo" << std::endl;
    }
};

int main() {
    void (Base:: * f1)() = &Base::goo;
    void (Base:: * f2)() = &Base::foo;
    void (Base:: * f3)() = &Base::hoo;

    printf("%d\n", &Base::goo);         // goo 의 index
    printf("%d\n", &Base::foo);         // foo 의 index
    printf("%d\n", &Base::hoo);         // hoo 의 index

    Base* p = new Derived;
    (p->*f1)();                         // p->goo(), goo(&p)
}
```
---
callback 과 함수 포인터 문제
```cpp
#include <iostream>

class Button {
    void(T::* handler)();
    Dialog* pDlg;

public:
    void setHandler(void(*f)()) {
        handler = f;
    }

    void click() {
        handler();                  // 버튼이 눌렸다는 사실을 외부에 전달 한다.
                                    // 흔히 "객체가 외부에 이벤트를 발생한다" 라고 표현
    }
};

void btn1Handler() {
    std::cout << "버튼 1 클릭" << std::endl;
}

int main() {
    Button b1, b2;
    b1.setHandler(&btn1Handler);    // 버튼에 callback 함수 등록
    b1.setHandler(&Dialog::Close);  // ??
    b1.click();                     // 사용자가 버튼을 클릭하면 이 함수가 호출된다고 가정.
}
```
---
모든 함수의 주소를 담을 수 있는 도구
 - C, C++ : 문법적으로는 없다.
 - C# : delegate 라는 문법
 - Objective-C : selector 라는 문법
 - C++11 : function<> 모든 함수의 주소를 담을 수 있다.
```cpp
#include <iostream>

void foo() {
    std::cout << "foo" << std::endl;
};
void goo(int a) {
    std::cout << "goo : " << a << std::endl;
};
void hoo(int a, int b) {
    std::cout << "hoo : " << a << " " << b << std::endl;
};

class Dialog {
public:
    void Close() {
        std::cout << "Dialog close" << std::endl;
    }
};

#include <functional>

void koo(int a, int b, int c, int d) {
    std::cout << a << " " << b << " " << c << " " << d << std::endl;
}

int main() {
    std::function<void()> f = &foo;
    f();                                    // ok... foo() 호출
    f = std::bind(&goo, 5);                 // 인자 고정
    f();                                    // goo(5);

    Dialog dlg;
    f = std::bind(&Dialog::Close, &dlg);    // 객체를 고정

    f = std::bind(&hoo, 1, 2);
    f();                                    // hoo(1, 2);

    std::function<void(int)> f1 = &goo;
    f1 = &goo;
    f1(5);                                  // goo(5);

    f1 = bind(&hoo, std::placeholders::_1, 3);
    f1(5);                                  // hoo(5, 3);

    std::function<void(int, int)> f2;
    f2 = bind(&koo, std::placeholders::_2, 2, 9, std::placeholders::_1);
    f2(6, 3);                               // koo(3, 2, 9, 6);
                                            // std::placeholders::_1, _2, _3 ... 
                                            // placeholder 로 인자를 전달 할 수 있다.
}
```