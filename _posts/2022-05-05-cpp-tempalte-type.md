---
published: true
title: "템플릿 타입"
date: 2022-05-05 18:00:02 +9:00
last_modified_at: 2022-05-05 18:00:05 +9:00
categories: cpp
---
```cpp
class AAA {
public:
//    static int DWORD;
    typedef int DWORD;
};

template<typename T>
void foo(T a) {
    T::DWORD *p;            //
    // 1. DWORD 는 T 의 static 멤버 data 이다.
    // 곱하기 p 를 하고 있다.
    // 2. DWORD 는 typedef 등 으로 만든 타입이다.
    // 포인터 변수 p를 선언하고 있다.

    typename T::DWORD *p;   // 템플릿 안의 타입을 꺼낼때는 반드시  typename 을 표기해야 한다.
    
    typename AAA::DWORD a;  //
                            // AAA 는 템플릿이 아니고 일반 타입이다.
    
}

int main() {
    AAA aaa;
    foo(aaa);
}
```
> error code
>-
---
모든 tempalte 기반 컨테이너에는 저장하는 타입이 있다.<p>
이때, 그 타입을 외부에서 알고싶을때.
```cpp
template<typename T> class list {
public:
    typedef T value_type;
};

list<int>::value_type n;    // 결국 n 은 int 타입 이 된다.

template<typename T> void print(T &v);

int main() {
    list<double> v;
    print(v);
}

template<typename T>
void print(T &v) {
    // T: list<double>
    // T::value_type: double
    typename T::value_type n1 = v.front();
    
    // c++11 의 도입으로 기존 소스를 보다 간단하게 표현 할 수 있다.
    // 결국 value_type 의 개념이 없어도 된다.
    auto n2 = v.front();
}
```