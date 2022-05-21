---
title: "위임 생성자"
date: 2022-05-21 18:02:09 +9:00
last_modified_at: 2022-05-21 18:02:10 +9:00
categories: cpp
---
위임 생성자: Deligating constructor
 - 본인의 다른 생성자를 호출하는 생성자
```cpp
#include <iostream>

class Point {
public:
    int x = 0;  // class 안에서 멤버변수를 상수로 바로 초기화 할 수 있다.
    int y = 0;  // java, c#처럼.
    int a{0};   // 가장 권장하는 초기화 스타일
    
    Point(int a, int b): x(a), y(b) {}
    
    Point(): Point(0, 0) {  // 생성자에서 다른 생성자를 호출하는 코드
        // this(0, 0);      // this 자체는 함수나 함수포인터가 아니다.
        // Point(0, 0);     // 생성자에서 다른 생성자 호출하는게 아닌 임시객체를 만드는 표현
    }
};

int main() {
    Point p;
    std::cout<<p.x<<std::endl;  // 0
}
```