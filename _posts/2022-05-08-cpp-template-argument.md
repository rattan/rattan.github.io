---
published: true
title: "템플릿 인자"
date: 2022-05-08 16:31:14 +9:00
last_modified_at: 2022-05-08 16:31:15 +9:00
categories: cpp
---
템플릿 인자
 - 타입
 - 정수형 상수(변수 안됨, 실수 안됨)
```cpp
template<typename T = int, int N = 10>
struct stack {
    T buff[N];
};

int main() {
    stack<int, 10> s1;
    int n = 10;
    stack<int, n> s2;   //
                        // 컴파일 시간 상수만 가능.
    
    stack<int> s3;      // N 은 10
    stack<> s4;         // 모든 인자를 디폴트 값으로
}
```
[]: 
> error code
>- [][]
---
 - template 인자로 template 을 보낼 수 있다.
```cpp
template<class T>
class list {};

template<typename T, template<typename> class C>
class stack {
    // C c; // error. C 는 template 이다.
    C<T> c; // ok.
};

int main() {
    // list st1;        // error. list 는 template 이다.
     list<int> s2;      // ok. list<int> 는 type 이다.
    
    stack<int, list> s;
}
```
---
이미 c++ 표준에 list 가 있다.<p>
그런데 stack 이 필요하게 되었다.
 1. stack 을 다시 만든다.
 2. list 를 재 사용 한다.
Adapter 디자인 패턴
 - 기존 클래스의 인터페이스(함수 이름) 를 변경해서 클라이언트가 요구하는 새로운 클래스를 만드는 패턴
 - c++ 은 템플릿과 인라인으로 성능저하 없이 만들 수 있다.
```cpp
#include <list>

template<typename T>
class stack: public std::list<T> {
public:
    inline void push(const T &a) {
        std::list<T>::push_back(a);
    }
    inline void pop() {
        std::list<T>::pop_back();
    }
    inline T& top() {
        return std::list<T>::back();
    }
};

// s/w 재 사용은 상속, 포함 으로 만들 수 있다.
// 포함 이 좋은 경우가 더 많다.
#include <deque>

template<typename T, typename C = std::deque<T>>
class stack2 {
    C st;
public:
    inline void push(const T& a) {
        st.push_back(a);
    }
    inline void pop() {
        st.pop_back();
    }
    inline T& top() {
        return st.back();
    }
};

#include <vector>
#include <stack>        // 이 헤더에 위의 stack 코드가 있다.

int main() {
    stack<int> st1;
    stack2<int, std::list<int>> st2;
    
    std::stack<int> st3;
    std::stack<int, std::vector<int>> st4; // vector<int> 를 stack 으로 바꾸어 달라.
    
    st4.push(10);
    st1.push_front(1);  // 사용자가 실수 할 수 있다.
    
}
```
---
```cpp
#include <iostream>
#include <list>
#include <vector>

template<typename T, template<typename, typename> class C>
class stack {
    C<T, std::allocator<T>> c;
public:
    void push(const T &a) {
        c.push_back(a);
    }
};

int main() {
    stack<int, std::list> s;
    s.push(10);
}
```
가변인자 템플릿
```cpp
#include <iostream>

// 가변인자 템플릿. c++11 기술
template<typename ... Types>
void foo(Types ... args) {}

// 가변인자 클래스 템플릿
template<typename ... Types>
class Test {};

int main() {
    foo(1);
    foo(1, 3.3);    // Types: int, double
                    // args: 1, 3.3
    Test<int> t1;
    Test<int, double> t2;
}
```
---
```cpp
#include <iostream>

// 가변인자 템플릿. c++11 기술
template<typename ... Types>
void goo(Types ... args) {
    std::cout<<"goo"<<std::endl;
}

void hoo(int a, double d, char c) {
    std::cout<<"hoo"<<std::endl;
}

void joo(int *a, double *d, char *c) {
    std::cout<<"joo"<<std::endl;
}

// Types: 여러개의 타입을 가지고 있는 타입의 집합
// args: 여러개의 값을 가지고 있는 인자 집합: parameter pack
template<typename ... Types>
void foo(Types ... args) {
    // parameter pack 안에 인자 개수를 알고 싶다면
    std::cout<<sizeof...(args)<<std::endl;
    std::cout<<sizeof...(Types)<<std::endl;
    
    // parameter pack  을 다른 함수로 전달하는 방법
    goo(args...);
    hoo(args...);
    joo(&args...);
}

int main() {
    foo(1, 3.3, 'a');   // 3
                        // 3
                        // goo
                        // hoo
                        // joo
}
```
