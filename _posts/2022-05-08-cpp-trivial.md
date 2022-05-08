---
published: true
title: "trivial"
date: 2022-05-08 15:55:55 +9:00
last_modified_at: 2022-05-08 15:55:57 +9:00
categories: cpp
---
자명한 생성자(trivial constructor): 아무 일도 하지 않는 생성자
 - 가상함수가 없다
 - 부모가 없거나 부모의 생성자가 trivial 하다
 - 객체형 멤버가 없거나 객체현 멤버의 생성자가 trivial 하다
 - 사용자가 만든 생성자가 없다
생성자는 "trivial" 하다.
```cpp
#include <iostream>

class Base {};

class A: public Base {
    Base b;
public:
    virtual void foo();
};

int main() {
    // A 의 생성자는 trivial 한가?
    A *p = (A*)malloc(sizeof(A));
    
    new(p) A;   // p메모리에 대해 생성자 호출
                // placement new
    
    p->foo();
}
```
---
```cpp
#include <iostream>
#include <type_traits>

// 모든 타입의 배열을 복사하는 strcpy() 의 일반화 버전을 만들어 보자
template<typename T>
inline void copy_type(T *d, T *s, int size) {
    if(!std::is_trivially_copyable<T>::value) {
        std::cout<<"복사 생성자가 하는 일이 있을때"<<std::endl;
        while(size--) {
            // new(d) T;    // 디폴트 생성자 호출
            new(d) T(*s);   // d 객체에 대해서 복사 생성자(*s 인자로) 호출
            ++d; ++s;
        }
    } else {
        std::cout<<"복사 생성자가 하는 일이 없을때"<<std::endl;
        memcpy(d, s, sizeof(T) * size);
    }
}

int main() {
    char s1[10] = "hello";
    char s2[10];
    
    copy_type(s2, s1, 10);
    std::cout<<s2<<std::endl;   // 복사 생성자가 하는 일이 없을때
                                // hello
}
```
---
trivial traits 를 만드는 일반적인 기술은 어렵다
```cpp
#include <iostream>

// android frameworks 에서는
template<typename T>
struct has_trivial_ctor {
    enum { value = false };
};

template<>
struct has_trivial_ctor<int> {
    enum { value = true };
};
template<>
struct has_trivial_ctor<double> {
    enum { value = true };
};

struct Point {
    int x, y;
};

template<>
struct has_trivial_ctor<Point> {
    enum { value = true };
};

int main() {
    if(has_trivial_ctor<int>::value) {}
}
```
---
