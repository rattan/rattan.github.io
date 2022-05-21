---
title: "일관된 초기화"
date: 
last_modified_at: 
categories: cpp
---
일관된 초기화: Uniform initialize
 - 변수의 종류에 상관없이 동일한 방법으로 초기화 할 수 있게 하자
```cpp
struct Point {
    int x, y;
};
class Complex {
    int re, im;
public:
    Complex(int a, int b): re(a), im(b) {}
};

int main() {
    int n1 = 0;
    int n2(2);
    int x[10] = {0};
    Point p = {1, 1};
    Complex c(0, 0);
    
    // 일관된 초기화
    // 모든 변수를 {} 로 초기화 한다.
    int n3{0};
    int x2[10]{0};
    Point p2{1, 1};
    Complex c2{0, 0};
    
    int n4 = 3.4;           // ok. c 와의 호환성 떄문에 c++ 도 지원하기로 했다.
    int n5{3.4};            // error.
    char ch{300};           // error. 300 은 1Byte 로 표현 할 수 없다.
    unsigned char ch2{200}; // ok.
}
```