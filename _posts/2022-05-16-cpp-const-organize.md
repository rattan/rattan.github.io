---
title: "const 정리"
date: 2022-05-16 19:03:15 +9:00
last_modified_at: 2022-05-16 19:03:17 +9:00
categories: cpp
---
1. const 위치: 아래 2개는 완전히 동일한 표현이다.
   - const int c1 = 10;
   - int const c2 = 10;
2. 상수와 비상수를 가르키는 포인터
```cpp
int main() {
    int n = 10;
    const int* p1 = &n; // *를 기점으로 보고 p1 을 따라가면 const int 가 있다.
                        // 상수를 가르키는 포인터 = &비상수

    const int c = 10;
    int* p2 = &c;       // error C2440: '초기화 중': 'const int *'에서 'int *'(으)로 변환할 수 없습니다.
                        // 원본 자체가 Read only 인데, 포인터를 통해서는 쓰기도 하겠다는 의미
                        // 비상수를 가르키는 포인터 = &상수 는 안됨.
    const int* p3 = &c; // 상수를 가르키는 포인터 = &비상수
}
```
---
```cpp
int main() {
    int n = 10;
    int n2 = 10;

    // 어느 메모리가 상수일까?
    const int* p1 = &n; // p1 을 따라가면 const 가 있다.
    int* const p2 = &n; // p2 를 따라가면 const 가 아니다.
                        // 하지만 p2 자체가 const 이다.
    p2 = &n2;           // error C3892: 'p2': const인 변수에 할당할 수 없습니다.
                        // 다시 대입할수 없다.
    *p2 = 20;           // p2 를 따라가면 const 가 아니다.

    // 아래 2줄은 동일한 표현이다.
    int const* p3 = &n; // p1 과 동일
    const int* p4 = &n;

}
```
---
this 의 상수성
```cpp
class Point {
    int x, y;
public:
    void set() {    // void set(Point* const this)
        // this 자체는 const 이지만 this 가 가르키는 곳은 const 가 아니다
        x = 0;      // this->x = 0;
        y = 0;      // this->y = 0;
        this = 0;   // error C2106: '=': 왼쪽 피연산자는 l-value이어야 합니다.
                    // this 포인터 자체는 상수 이다.
    }

    void print() const {  // void print(const Point *const this)
        // this 도 const, this 가 가르키는 곳도 const
        x = 10; // error C3490: 'x'은(는) const 개체를 통해 액세스되고 있으므로 수정할 수 없습니다.
                // this->x = 10;
    }
};
```
---
```cpp
int main() {
    int n = 10;
    
    const int* p1 = &n;     // p1 은 const 가 아님. p1 을 따라가면 const
    int* const p2 = &n;     // p2 는 const p2 를 따라가면 const 아님

    const int*& r1 = p1;    // 상수를 가르키는 포인터 참조
    int* const& r2 = p2;    // 상수 포인터(포인터 자체가 상수) 의 참조
}
```