---
published: true
title: "클래스 템플릿"
date: 2022-05-05 17:23:36 +9:00
last_modified_at: 2022-05-05 17:23:39 +9:00
categorise: cpp
---
```cpp
template<typename T>
class stack {
    T buff[100];
    int index;
public:
    // 생성자
    stack() {}      // 정확한 생성자 표현
    //stack<T>() {} // 일부 컴파일러는 이 표현도 허용
                    // 하지만 stack() 이 정확한 표현
    // 복사 생성자
    //stack(const stack &s) {}  // 클래스 내부에서는 이 표현도 허용
                                // 단 외부 구현시에는 이 표현으 error.
    stack(const stack<T> &s) {} // 정확한 표현
    
    void push(T a);             // 멤버 함수의 외부 구현
    
    static int count;           // static 멤버 data 의 외부 구현
    
    template<typename U>        // 멤버함수 템플릿
    T foo(U a);
};

template<typename T> template<typename U>
T stack<T>::foo(U a) {          // 클래스 템플릿의 멤버함수 템플릿을 외부 구현 한 모습
    // ...
}

template<typename T>
int stack<T>::count = 0;        // static 멤버 data 의 외부 구현

template<typename T>
void stack<T>::push(T a) {
    
}

int main() {
    stack s1;                   // error.
                                // stack 은 타입이 아니라 틀(template)이다.
    stack<int> s2;              // ok. stack<int> 는 타입니다.
}
```
[stack s1]: 
> error code
>- [stack s1][]
---
왜 클래스 템플릿의 멤버함수 템플릿을 사용하는가?
```cpp
#include<iostream>

template<typename T>
class complex {
    T re;
    T im;
public:
    // zero initialize: T a = T();
    // T 가 표준 타입이거나 포인터 이면 0 으로 초기화
    // T 가 사용자 타입이면 디폴트 생성자로 초기화
    complex(T r = T(), T i = T()): re(r), im(i) {}
    
    // 일반화 된(generic) 복사 생성자
    // U가 T로 복사(대입) 될 수 있다면
    // complex<U> 는 complex<T> 로 복사(대입) 될 수 있어야 한다.
    // 일반환 된 복사생성자(대입 연산자) 가 필요하다.
    template<typename U> complex(const complex<U> &c);
    
    // 모든 타입의 complex 들은 서로 private에 접근가능해야 한다.
    // friend void foo(); friend 함수.
    template<typename>
    friend class complex;
    
    void t() {
        std::cout<<re<<" "<<im<<std::endl;
    }
};

// 일반화 된 복사생성자의 외부 구현
template<typename T> template<typename U>
complex<T>::complex(const complex<U> &c): re(c.re), im(c.im){
    std::cout<<"일반화 된 복사생성자"<<std::endl;
}

int main() {
    complex<int> c1(1, 2);
    complex<int> c2 = c1;       // 복사생성자 사용: 디폴트 복사생성자
    
    complex<double> c3 = c1;    // 일반화 된 복사생성자
}
```
---
```cpp
class Animal {};
class Dog: public Animal {};

int main() {
    Dog *p1 = new Dog;
    Animal *p2 = p1;    // 부모 타입의 포인터는 자식의 주소를 담을 수 있다.
    
    // 요즘은 포인터를 직접 사용하는것 보다 스마트 포인터를 많이 사용 한다.
    // e.g. sp: android framework 의 스마트 포인터
    sp<Dog> p3 = new Dog();
    sp<Animal> p4 = p3;
    p4 = p3;
    if(p4 == p3) {
    }
    // 스마트 포인터에는 반드시 복사, 대입, ==, != 가 template 버전으로 있어야 한다.
}
```
[StrongPointer.h]: https://android.googlesource.com/platform/system/core/+/refs/heads/master/libutils/include/utils/StrongPointer.h
>[StrongPointer.h][]