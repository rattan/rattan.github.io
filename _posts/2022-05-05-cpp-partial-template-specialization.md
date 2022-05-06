---
published: true
title: "템플릿 부분 전문화"
date: 2022-05-05 20:12:57 +9:00
last_modified_at: 2022-05-05 20:12:58 +9:00
categories: cpp
---
템플릿 부분 전문화
```cpp
#include <iostream>

template<typename T>
class stack {
    T buff[100];
public:
    void push() {
        std::cout<<"T"<<std::endl;
    }
};

// partil specialization, 부분 전문화, 부분 특화
template<typename T>
class stack<T*> {
    T buff[100];
public:
    void push() {
        std::cout<<"T*"<<std::endl;
    }
};

// specialization, 전문화
template<>
class stack<char*> {
    char* buff[100];
public:
    void push() {
        std::cout<<"char*"<<std::endl;
    }
};

int main() {
    stack<int> s1;
    s1.push();      // T
    stack<int*> s2;
    s2.push();      // T*
    stack<char*> s3;
    s3.push();      // char*
}
```
---
template meta programming: 컴파일 시에 연산을 수행 하는것
 - 컴파일 시간 재귀를 활용 한다.
 - 재귀의 종료를 위해서 전문화 문법이 활용된다.
 - 메타 함수 라고 한다.
```cpp
#include <iostream>

template<int N>
struct Factorial {
    enum { value = N * Factorial<N-1>::value };
};

template<>
struct Factorial<1> {
    enum { value = 1 };
};

template<int N>
struct Binary {
    typedef char bianry_digit_not_allow_2_to_9[N % 10 > 1 ? -1: 0];
    enum { value = (N & 1) | Binary<(N/10)>::value<<1 };
};

template<>
struct Binary<1> {
    enum { value = 1 };
};

template<>
struct Binary<0> {
    enum { value = 0 };
};

int main() {
    int n = Factorial<5>::value;    // 5 * F<4>::v
                                    // 5 * 4 * F<3>::v
                                    // 5 * 4 * 3 * F<2>::v
                                    // 5 * 4 * 3 * 2 * F<1>::v
                                    // 5 * 4 * 3 * 2 * 1
    std::cout<<n<<std::endl;        // 120
    
    int a = 5;
    std::cin>>a;
    int n2 = Factorial<a>::value;   // error
                                    // 템플릿 인자는 컴파일시간 상수만 가능
    
    int n3 = Binary<101>::value;
    std::cout<<n3<<std::endl;       // 5
}
```
작성중 ~2022-05-05 20:12:36 +9:00
