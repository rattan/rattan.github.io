---
published: true
title: "임시 객체"
date: 2022-05-10 21:07:49 +9:00
last_modified_at: 2022-05-10 21:07:51 +9:00
categories: cpp
---
 - Point p;
   - 이름있는 객체 p
   - 블럭을 벗어나기 전 까지 생존
 - Point();
   - 이름 없는 임시 객체
   - 현재 문장에서만 유효 하다.
```cpp
#include <iostream>

class Point {
    int x, y;
public:
    Point() {
        std::cout << "생성자" << std::endl;
    }
    ~Point() {
        std::cout << "소멸자" << std::endl;
    }
    Point(const Point&) {
        std::cout << "복사 생성자" << std::endl;
    }
};

// 함수 인자와 임시객체
void foo(Point p) {}

int main() {
    Point p;        // 생성자
    foo(p);         // 복사 생성자
                    // 소멸자

    // 값 타입의 함수 인자로 전달하기 위한 객체라면 임시 객체가 좋다.
    foo(Point());   // 생성자
                    // 소멸자
}                   // 소멸자
```
---
```cpp
#include <iostream>

class Point {
    int x, y;
public:
    Point() {
        std::cout << "생성자" << std::endl;
    }
    ~Point() {
        std::cout << "소멸자" << std::endl;
    }
    Point(const Point&) {
        std::cout << "복사 생성자" << std::endl;
    }
};

// 함수 리턴과 임시 객체
// 값 타입으로 리턴하는 함수는 임시 객체를 만들게 된다.
Point foo() {
    Point p2;
    std::cout << "foo" << std::endl;
    return p2;
    
    // 리턴 용도로만 객체를 사용한다면
    // return Point(); 와 같이 만드는게 좋다.
                        // 임시 객체를 만들면서 리턴.
                        // Return Value Optimization(RVO) 라는 c++ IDioms
                        // 요즘 컴파일러는 NRVO(Named RVO) 를 지원 한다.
}

int main() {
    Point p1;                           // 생성자
    std::cout << "AAA" << std::endl;    // AAA
    p1 = foo();                         // 생성자
                                        // foo
                                        // 소멸자
    std::cout << "BBB" << std::endl;    // BBB
}                                       // 소멸자
```
---
```cpp
#include <iostream>

struct Point {
    int x, y;
};
Point p = { 1,1 };

//Point foo();  // 값 리턴: 임시 객체 리턴, lvalue 가 될 수 없다.
Point& foo() {  // 참조 리턴: 임시객체 생성안됨, lvalue 가 될 수 있다.
    return p;   // 단 지역변수의 참조는 절대 안된다.
}

int main() {
    // 임시 객체는 lvalue 가 될 수 없다.
    // Point foo(); 로 만들어진 임시객체인 경우 error.
    foo().x = 10;                   // p.x = 10;

    std::cout << p.x << std::endl;  // 10
}
```
---
```cpp
#include <iostream>

template<typename T>
class stack {
    T buff[100];
    int index;
public:
    stack() : index(-1) {}
    
    // T 가 크기가 큰 객체 일 수 있다.
    // call by value 는 동일 객체를 메모리에 한번 더 생성한다.
    // void push(T a) { buff[++index] = a; }
    // const T& 로 넘기는게 좋다.
    void push(const T& a) {
        buff[++index] = a;
    }

    // pop() 이 제거와 리턴을 동시에 하면 최적화 할 수 없다.
    // 임시 객체를 막을 수 없다.
    // T pop() { return buff[index--]; }
    // 제거와 리턴은 분리 되어야 한다.
    void pop() {
        --index;            // 제거만
    }
    T& top() {
        return buff[index]; // 리턴만
    }
};

int main() {
    stack<int> s;
    s.push(10);
    s.push(20);
    std::cout << s.top() << std::endl;  // 20
    s.pop();
    std::cout << s.top() << std::endl;  // 10
}
```
c++ 에서 template 기반의 컨테이너(list, stack, vector 등) 의 모든 함수는 제거와 리턴을 동시에 하지 않는다.
 - 참조 리턴을 통해 임시 객체를 제거 하기 위해
 - 예외 안전성의 강력 보장을 위해