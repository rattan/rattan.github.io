---
published: true
title: "상수 함수"
date: 2022-05-03 21:40:38
last_modified_at: 2022-05-03 21:40:38
categories: cpp
---
```cpp
#include <iostream>

class Point {
public:
    int x, y;
    Point(int a = 0, int b = 0) :x(a), y(b) {}

    void set(int a, int b) {
        x = a;
        y = b;
    }

    void print() const {
        x = 10;     // error C3490: 'x'은(는) const 개체를 통해 액세스되고 있으므로 수정할 수 없습니다.
        std::cout << x << ", " << y << std::endl;
    }
};

int main() {
    const Point p(0, 0);

    p.x = 10;       // error C3892: 'p': const인 변수에 할당할 수 없습니다.
    p.set(10, 10);  // error C2662: 'void Point::set(int,int)': 'this' 포인터를 'const Point'에서 'Point &'(으)로 변환할 수 없습니다.
    p.print();      // 호출되게 하려면 print() 는 상수함수 이어야 한다.
                    // 상수 객체는 상수함수만 호출 할 수 있다.
}
```
>error code
>- [C3490 식이 수정할 수 있는 lvalue여야 합니다.](https://docs.microsoft.com/ko-kr/cpp/error-messages/compiler-errors-2/compiler-error-c3490){:target="_blank"}
>- [C3892 식이 수정할 수 있는 lvalue여야 합니다.](https://docs.microsoft.com/ko-kr/cpp/error-messages/compiler-errors-2/compiler-error-c3892){:target="_blank"}
>- [C2662 'void Point::set(int,int)': 'this' 포인터를 'const Point'에서 'Point &'(으)로 변환할 수 없습니다.](https://docs.microsoft.com/ko-kr/cpp/error-messages/compiler-errors-2/compiler-error-c2662){:target="_blank"}
---
 - 상수 함수는 필수 이다.
 - 객체의 상태를 변경하지 않는 모든 멤버함수는 반드시 상수함수로 만들어야 한다.
```cpp
class Rect {
    int x, y, w, h;
public:
    int getArea() const {
        return w * h;
    }
};

void foo(const Rect& r) {   // c++ 기본 기법: call by value 대신 const& 가 좋다.
    int n = r.getArea();
}

int main() {
    Rect r;                 // 초기화 되었다고 가정하자.

    int n = r.getArea();    // ok.

    foo(r);
}
```
---
논리적 상수성
 - 외부에서 바라볼 때는 상수함수가 되어야 하지만 내부적으로는 멤버변수의 값을 변경해야 하는 문제
 - 변하는 멤버는 mutable 로
```cpp
#include <iostream>

class Point {
    int x, y;
    mutable char cache[32];
    mutable bool cache_valid;
public:
    Point(int a = 0, int b = 0) : x(a), y(b), cache_valid(false) {}

    char* toString() const {    // 객체의 상태를 문자열로 반환하는 함수: java, c# 에 있는 개념
        if (cache_valid == false) {
            sprintf_s(cache, "%d %d", x, y);
            cache_valid = true;
        }
        return cache;
    }
};

int main() {
    Point p(1, 2);
    std::cout << p.toString() << std::endl;
    std::cout << p.toString() << std::endl;
}
```
---
- 변하지 않는것과 변하는 것은 분리되어야 한다.
- 상수함수에서 변해야 하는 것이 있다면 별도의 구조체로 분리한다.
```cpp
#include <iostream>

struct Cache {          // 변하는 부분을 구조체로 빼낸다.
    char data[32];
    bool valid;
};

class Point {
    int x, y;
    Cache* pCache;
public:
    Point(int a = 0, int b = 0) : x(a), y(b) {
        pCache = new Cache;
        pCache->valid = false;
    }

    char* toString() const {
        if (pCache->valid == false) {
            sprintf_s(pCache->data, "%d %d", x, y);
            pCache->valid = true;
        }
        return pCache->data;
    }

    ~Point() {
        delete pCache;
    }
};

int main() {
    Point p(1, 2);
    std::cout << p.toString() << std::endl;
    std::cout << p.toString() << std::endl;
}
```