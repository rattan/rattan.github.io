---
published: true
title: "traits"
date: 2022-05-06 19:18:07 +9:00
last_modified_at: 2022-05-06 19:18:09 +9:00
categories: cpp
---
```cpp
#include <iostream>

// T 를 받아 출력하는 함수를 생각해 보자.
template<typename T>
void printv(T a) {
    std::cout<<a<<std::endl;
}

// T타입의 종류에 따라 다르게 동작하게 할 필요가 있다.
template<typename T>
void printv2(T a) {
    if(T is Pointer) {
        std::cout<< a <<", "<< *a <<std::endl;
    } else {
        std::cout<< a <<std::endl;
    }
}

int main() {
    int n = 3;
    double d = 3.3;
    
    printv(n);
    printv(d);
    
    printv2(&d);
}
```
---
Traits 기술
 - T 의 다양한 속성(특질)을 컴파일 시간에 알아내는 기술
 - T 에 대해서 조사 할 수 있는 메타 함수
만드는 방법
 - primary template: false 리턴 (value = false)
 - 부분 전문화 버전: true 리턴 (value = true)
```cpp
#include <iostream>

// primary template
template<typename T>
struct IsPointer {
    enum { value = false };
};

// 부분 전문화버전
// T 가 포인터인 경우 value 를 true 로 만든다.
template<typename T>
struct IsPointer<T*> {
    enum { value = true };
};

// 컴파일러의 동작 방식
// 1. T를 결정해서 함수 생성
// 2. 생성된 함수 컴파일: 문법 오류 확인
// 3. 최전화 해서 실행되지 않을 코드 제거
template<typename T>
void foo(T a) {
    // 컴파일 시간에 if 내의 true/false 가 결정된돠.
    // 즉, if/else 둘중 하나의 코드는 인스턴스화 된 코드에서 전혀 필요가 없다.
    if(IsPointer<T>::value) {
        std::cout<<"포인터"<<std::endl;
    } else {
        std::cout<<"포인터 아님"<<std::endl;
    }
}

int main() {
    int n = 0;
    foo(n);     // 포인터 아님
    foo(&n);    // 포인터
}
```
---
```cpp
#include <iostream>

// primary template
template<typename T>
struct IsPointer {
    enum { value = false };
};

// 부분 전문화버전
// T 가 포인터인 경우 value 를 true 로 만든다.
template<typename T>
struct IsPointer<T*> {
    enum { value = true };
};

// 포인터인 경우를 처리하는 함수
template<typename T>
void printv_imp(T a, YES) {
    std::cout<< a << ", " << *a <<std::endl;
}
// 포인터가 아닌경우
template<typename T>
void printv_imp(T a, NO) {
    std::cout<< a <<std::endl;
}

template<typename T>
void printv(T a) {
    // if 문은 실행시간이다. 그래서 아래 코드는 실행시간에 함수가 선택 된다.
    // 그래서 컴파일러는 YES버전 과 NO버전의 모든 함수 템플릿을 인스턴스화 하게 된다.
    // 결국 int 인 경우 불필요한 YES 버전 생성
    if(T is Pointer) print_imp(a, YES);
    else print_imp(a, NO);
    
    // 함수 오버로딩은 컴파일 시간 함수 선택
    // 선택되지 않은 템플릿은 인스턴스화 되지 않는다.
    // 즉 int 인 경우 YES 버전은 컴파일 되지 않는다.
    printv_imp(a, T is Pointer);
}

int main() {
    int n = 3;
    printv(n);
}
```
---
0, 1 로 함수 오버로딩이 되게 하는 기술
 - 모든 int 형 상수를 타입으로 만드는 시스템
 - Modern c++ 저자인  Andrei Alexander 가 2000년도에 만든 기술
```cpp
#include <iostream>

// 컴파일 시간 결정된 모든 상수는 함수 오버로딩에 활용 할 수 있다.
template<int N>
struct int2type {
    enum { value = N };
};

template<typename T>
void foo(T a) {
    std::cout<<a<<std::endl;
}

int main() {
    // 0, 1 은 같은 타입이다.
    // foo(0), foo(1) 은 같은 함수를 호출한다.
    foo(0);
    foo(1);
    
    // t0 와 t1 은 다른 타입이라.
    // foo(t0), foo(t1) 은 다른 함수를 호출 한다.
    int2type<0> t0;
    int2type<1> t2;
    foo(t0);
    foo(t2);
}
```
---
```cpp
#include <iostream>

template<int N>
struct int2type {
    enum { value = N };
};

// int2type 의 타입을 결정할 값을 value 로 만든다.
template<typename T>
struct IsPointer {
    enum { value = 0 };
};
template<typename T>
struct IsPointer<T*> {
    enum { value = 1 };
};

// 포인터인 경우를 처리하는 함수, int2type<1> 타입으로 오버로딩
template<typename T>
void printv_imp(T a, int2type<1>) {
    std::cout<< a << ", " << *a <<std::endl;
}
// 포인터가 아닌경우, int2type<0> 타입으로 오버로딩
template<typename T>
void printv_imp(T a, int2type<0>) {
    std::cout<< a <<std::endl;
}

template<typename T>
void printv(T a) {
    // int2type 으로 만들어진 함수객체를 넘겨 오버로딩 된 함수를 호출하게 만든다.
    printv_imp(a, int2type<IsPointer<T>::value>());
}

int main() {
    int n = 3;
    printv(n);  // 3
    printv(&n); // 0x7ffeefbff44c, 3
}
```
---
c++표준 위원회는 int2type 을 발전시켜 아래 템플릿을 제공한다.
```cpp
template<typename T, T N>
struct integral_constant {
    // enum { value = N };
    
    // static const 는 클래스 안에서 바로 초기화 가능 하다: 2000년 초 추가된 문법
    // 그래서 요즘음 enum대신 사용한다.
    static const T value = N;
};

integral_constant<int, 0> n0;
integral_constant<short, 0> s0;

// true/false: 참, 거짓을 나타내는 값: 같은 타입
// true_type/false_type: 참, 거짓을 나타내는 타입: 다른 타입
typedef integral_constant<bool, true> true_type;
typedef integral_constant<bool, false> false_type;

#include <iostream>
#include <type_traits>  // 여기 위의 템플릿들미 이미 만들어져 있다.

// 이제 is_poiner 등을 만들때 아래와 같이 만들 수 있다.
// 상속의 개념이 추가되고, value 는 부모로 부터 물려받는다.
template<typename T> struct is_pointer: std::false_type{};
template<typename T> struct is_pointer<T*>: std::true_type{};

template<typename T> void printv_imp(T a, std::true_type) {
    std::cout<< a << ", " << *a <<std::endl;
}
template<typename T> void printv_imp(T a, std::false_type) {
    std::cout<< a <<std::endl;
}
template<typename T> void printv(T a) {
    printv_imp(a, is_pointer<T>());
}

int main() {
    int n = 3;
    printv(n);  // 3
    printv(&n); // 0x7ffeefbff44c, 3
}
```
---
```cpp
#include <iostream>
#include <type_traits>  // 여기에 integral_const<> 와
                        // is_pointer<> 등 100 여개의 traits 가 있다.

void check(std::true_type) {
    std::cout<<"포인터"<<std::endl;
}

void check(std::false_type) {
    std::cout<<"포인터 아님"<<std::endl;
}

template<typename T>
void foo(const T& a) {
    // traits 사용 방법
    // value 를 꺼내서 조사
    // 단점: 포인터가 아닌 else 에서 *a 등의 표현이 있으면 안된다.
    if(std::is_pointer<T>::value) {
        std::cout<<"포인터"<<std::endl;
    } else {
        std::cout<<"포인터 아님"<<std::endl;
    }
    
    // 함수 오버로딩
    // 장점: 포인터와 포인터 아닌 경우가 별개의 함수로 처리된다.
    // *a 등의 표현 가능
    check(std::is_pointer<T>());
}

int main() {
    int n = 10;
    foo(n);     // 포인터 아님
    foo(&n);    // 포인터
}
```
---
```cpp
#include <iostream>
#include <type_traits>

int a;
int *p;
int x[10];  // int[10] 가 x 의 타입이다: T[N]

//
template<typename T>
struct IsArray: std::false_type {
    static const int size = -1; // 배열이 아니므로
};

template<typename T, int N>
struct IsArray<T[N]>: std::true_type {
    static const int size = N;
};

template<typename T>
void foo(const T &a) {
    if(IsArray<T>::value) {
        std::cout<<"배열, 크기는 "<<IsArray<T>::size<<std::endl;
    } else {
        std::cout<<"배열 아님"<<std::endl;
    }
}

int main() {
    int x[10];
    int y;
    foo(x); // 배열, 크기는 10
    foo(y); // 배열 아님
}
```
---
 traits 종류
```cpp
#include <iostream>
#include <type_traits>

// T 의 속성 질의
template<typename T>
void foo(const T &a) {
    if(std::is_array<T>::value) {
        std::cout<<"배열, 크기는 "<<std::extent<T, 1>::value<<std::endl;
    }
}

// T 타입의 변형
template<typename T>
void goo(const T &a) {
    typename std::remove_pointer<T>::type n;
//    typename std::add_pointer<T>::type n;
    std::cout<<typeid(n).name()<<std::endl;
}

int main() {
    int x[10][20];
    foo(x);     // 배열, 크기는 20
    int n;
    goo(&n);    // int
}
```
