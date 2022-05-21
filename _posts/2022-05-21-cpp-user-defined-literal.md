---
tile: "사용자 정의 상수"
date: 2022-05-21 17:45:07 +9:00
last_modified_at: 2022-05-21 17:45:08 +9:00
categories: cpp
---
사용자 정의 상수: 상수 값으로 사용자 정의 타입을 만들 수 있다.
 - c++ 기본 상수
   - 1.0f
   - 2l
   - "string"s: stl
 - 사용자 정의 상수
   - 1_m
   - 2.0_dd
   - "my string"_p
```cpp
#include <iostream>

class Meter {
    int value;
public:
    Meter(int n): value(n) {}
    void print() const {
        std::cout<<value<<std::endl;
    }
};

// 사용자 정의 상수를 만들때 인자 타입은 미리 정의되어 있다.
// unsigned long long
// const char*
// unsigned long double
Meter operator"" _m(unsigned long long sz) {    // 사용자 정의 상수 이름앞에 underscore(_) 가 붙어야 한다.
    return Meter(sz);                           // 표준 라이브러리만 _없이 사용할 수 있다.
}

int main() {
    float f = 3.4f; // f, s, l
    Meter meter = 3_m;  // operator"" _m(3)
    meter.print();
}
```
---
```cpp
#include <iostream>
#include <string>
#include <complex>

void foo(const char* s) {
    std::cout<<"char*"<<std::endl;
}
void foo(std::string s) {
    std::cout<<"string"<<std::endl;
}

int main() {
    foo("hello");               // char*
    foo(std::string("hello"));  // string
    
    // stl 상수 사용 선언을 해야 한다.
    using std::string_literals::operator""s;
    foo("hello"s);              // string
    
    using std::complex_literals::operator""i;
    std::complex<int> c1(3);    // 3 + 0i
    std::cout<<c1<<std::endl;   // (3, 0)
    
    std::complex<int> c2(0, 3); // 0 + 3i
    std::cout<<c2<<std::endl;   // (0, 3)
    
    std::complex<int> c3(3i);   // 0 + 3i
    std::cout<<c3<<std::endl;   // (0, 3)
}
```