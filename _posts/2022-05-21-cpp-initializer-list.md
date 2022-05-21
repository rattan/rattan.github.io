---
title: "initializer list"
date: 2022-05-21 18:36:45 +9:00
last_modified_at: 2022-05-21 18:36:46 +9:00
categories: cpp
---
```cpp
#include <iostream>

void foo(std::initializer_list<int> e) {
    auto p = e.begin();     // begin(e);
    while(p != e.end()) {   // end(e);
        std::cout<<*p<<" ";
        ++p;
    }
    std::cout<<std::endl;
}

int main() {
    std::initializer_list<int> e1{1, 2, 3, 4, 5};
    std::initializer_list<int> e2 = {1, 2, 3, 4, 5};
    foo(e1);        // 1 2 3 4 5
    foo({1, 2, 3}); // 1 2 3
                    // c++98: error. c++11 부터 지원
}
```
---
생성자와 initialize list
```cpp
#include <iostream>
#include <vector>

class Point {
    int x, y;
public:
    Point(int a, int b) {
        std::cout<<"int, int"<<std::endl;
    }
    Point(std::initializer_list<int> e) {
        std::cout<<"initialize list"<<std::endl;
    }
};

int main() {
    Point p1(1, 2);     // int, int
    Point p2({1, 2});   // initialize list
    Point p3{1, 2};     // initialize list
                        // 없으면 int, int 호출
    
    Point p4(1, 2, 3);  // error.
    Point p5{1, 2, 3};  // ok
    Point p6 = {1, 2};  // ok
    
    // 1 ~ 10 까지로 초기화 된 vector 만들기
    std::vector<int> v = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
}
```