---
title: "this call 1"
date: 2022-04-29 20:46:25
categories: c++
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
    void Close() { std::cout<<"Dialog close"<<std::endl; }
};

void foo() { std::cout<<"foo"<<std::endl; }

int main() {
    void(*f1)() = &foo;                     // ok.
    void(*f2)() = &Dialog::Close;           // error C2440: '초기화 중': 'void (__cdecl Dialog::* )(void)'에서 'void (__cdecl *)(void)'(으)로 변환할 수 없습니다.

    void(Dialog::*f3)() = &Dialog::Close;   // ok. 멤버함수 포인터를 만드는 법
    f3();                                   // error C2064: 항은 0개의 인수를 받아들이는 함수로 계산되지 않습니다.
                                            // this 가 없다.

    Dialog dlg;
    dlg.f3();                               // error C2039: 'f3': 'Dialog'의 멤버가 아닙니다.
                                            // dlg.Close() 즉 f3(&dlg)
                                            // 하지만 이 경우 컴파일러가 f3 이라는 멤버를 찾게되어 error

    (dlg.*f3)();                            // f3 은 함수포인터 이므로 *f3 하면 함수가 된다.
                                            // .*연산자 우선 순위를 함수호출() 보다 높여야 한다.

    Dialog *pDlg = &dlg;
    (pDlg->*f3)();                          // 포인터인 경우 arrow operator 로 호출할 수 있다.
}
```
>error code
>- [C2440 "void(Dialog::*)()"형식의 값을 사용하여 "void(*)()"형식의 엔터티를 초기화할 수 없습니다.](https://docs.microsoft.com/ko-kr/cpp/error-messages/compiler-errors-1/compiler-error-c2440)
>- [C2064 명백한 호출의 괄호 앞에 오는 식에는 함수 (포인터) 형식이 있어야 합니다.](https://docs.microsoft.com/ko-kr/cpp/error-messages/compiler-errors-1/compiler-error-c2064)
>- [C2039 클래스 "Dialog"에 "f3" 멤버가 없습니다.](https://docs.microsoft.com/ko-kr/cpp/error-messages/compiler-errors-1/compiler-error-c2039)