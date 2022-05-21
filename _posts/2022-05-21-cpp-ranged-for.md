---
title: "ranged for"
date: 2022-05-21 17:08:45 +9:00
last_modified_at: 2022-05-21 17:08:47 +9:00
categories: cpp
---
```cpp
#include <iostream>
#include <vector>

int main() {
    int x[10] = {1,2,3,4,5,6,7,8,9,10};
    std::vector<int> v(x, x+10);
    
    for(auto n: v) {        // ranged for 문
        std::cout<<n<<" ";  // 1 2 3 4 5 6 7 8 9 10
    }
    std::cout<<std::endl;
    
    // 위 코드를 컴파일 하면 아래 코드가 된다.
    for(auto p = begin(v); p != end(v);++p) {
        auto n = *p;
        std::cout<<n<<" ";  // 1 2 3 4 5 6 7 8 9 10
    }
    std::cout<<std::endl;
}
```
---
사용자 정의 타입에 대해 ranged-for 을 사용하려면 begin()/end() 를 만들면 된다.
```cpp
#include <iostream>

struct Point3D {
    int x, y, z;
};

int* begin(Point3D& p) {
    return &(p.x);
}
int* end(Point3D& p) {
    return &(p.z) + 1;
}

int main() {
    Point3D pt = {1, 2, 3};
    for(auto n: pt) {
        std::cout<<n<<std::endl;    // 1
                                    // 2
                                    // 3
    }
}
```
---
[begin end](/cpp/cpp-begin-end/) 에서만든 show()함수: c++98 스타일<p>
c++11의 새로운 문법을 사용하면 훨씬 간단하게 코드를 만들 수 있다.
```cpp
#include <iostream>
#include <vector>

template<typename T>
void show(T& c) {
    for(auto n: c) {
        std::cout<<n<<" ";
    }
    std::cout<<std::endl;
}

int main() {
    int x[10] = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
    std::vector<int> v(x, x+10);
    
    show(x);    // 1 2 3 4 5 6 7 8 9 10
    show(v);    // 1 2 3 4 5 6 7 8 9 10
}
```