---
published: true
title: "nulltpr"
date: 2022-05-11 22:01:34 +9:00
last_modified_at: 2022-05-11 22:01:36 +9:00
categories: 
---
nullptr: c++11 의 새로운 개념
```cpp
#include <iostream>

void foo(int n) {
    std::cout << "int" << std::endl;
}

void foo(void* p) {
    std::cout << "void*" << std::endl;
}

void goo(char* p) {
    std::cout << "char*" << std::endl;
}

int main() {
    foo(0);         // int
                    // 0 은 정수 이다.
    foo((void*)0);  // void*
                    // 0 번지의 void*

#define NULL (void*)0
    foo(NULL);
    goo(NULL);      // error C2664: 'void goo(char *)': 인수 1을(를) 'void *'에서 'char *'(으)로 변환할 수 없습니다.
                    // c 에서는 ok. void* => 다른 포인터 로 암시적 변환 허용
                    // c++ 에서는 error.

    int* p1 = 0;    // 0 은 정수 이지만 포인터로 암시적 형 변환이 된다.
    int* p2 = 10;   // error C2440: '초기화 중': 'int'에서 'int *'(으)로 변환할 수 없습니다.
                    // 10 은 포인터로 형 변환 되지 않는다.
}
```
[C2664 "void *" 형식의 인수가 "char *" 형식의 매개 변수와 호환되지 않습니다.]: https://docs.microsoft.com/ko-kr/cpp/error-messages/compiler-errors-2/compiler-error-c2664
[C2440 "int" 형식의 값을 사용하여 "int *" 형식의 엔터티를 초기화할 수 없습니다.]: https://docs.microsoft.com/ko-kr/cpp/error-messages/compiler-errors-1/compiler-error-c2440
> error code
>- [C2664 "void *" 형식의 인수가 "char *" 형식의 매개 변수와 호환되지 않습니다.][]
>- [C2440 "int" 형식의 값을 사용하여 "int *" 형식의 엔터티를 초기화할 수 없습니다.][]
---
```cpp
#include <iostream>

void foo(int n) {
    std::cout << "int" << std::endl;
}

void foo(void* p) {
    std::cout << "void*" << std::endl;
}

void goo(char* p) {
    std::cout << "char*" << std::endl;
}

// c/c++ 간의 차이점 때문에 NULL 은 아래와 같이 만들어져 있다.
#ifdef __cplusplus
#define NULL 0
#else
#define NULL (void*)0
#endif

int main() {
    goo(NULL);
    foo(NULL);  // int
}
```
---
```cpp
#include <iostream>

void foo(int n) {
    std::cout << "int" << std::endl;
}

void foo(void* p) {
    std::cout << "void*" << std::endl;
}

void goo(char* p) {
    std::cout << "char*" << std::endl;
}

int main() {
    int* p = nullptr;   // 포인터 0 을 나타내는 상수
    char* p2 = nullptr;

    int n = nullptr;    // error C2440: '초기화 중': 'nullptr'에서 'int'(으)로 변환할 수 없습니다.
                        // 포인터 0 이지 정수 0 이 아니다.

    foo(nullptr);       // void*
    goo(nullptr);       // char*
}
```
---
nullptr 의 타입: nulltpr_t
```cpp
#include <iostream>

int main() {
    nullptr_t n = nullptr;
    int* p = nullptr;   // nullptr_t => 모든 포인터 타입 으로 암시적 변환
    //int a = nullptr;  // nullptr_t 은 int 로 암시적 형변환 되지 않는다.
    bool b(nullptr);    // nullptr 은 암시적으로 false 로 변환 된다.
    bool b2 = nullptr;  // 대입은 error.
    if (nullptr) {      // 그래서 조건 식에서 사용 할 수 있다.
    }
}
```
---
```cpp
#include <iostream>

void foo(int n) {
    std::cout << "foo" << std::endl;
}
void goo(char* p) {
    std::cout << "goo"<<std::endl;
}
void hoo(int a, int b) {
    std::cout << "hoo" << std::endl;
}
void koo() {
    std::cout << "koo" << std::endl;
}

// 함수가 실행되는 시간을 확인하는 도구를 만들어 보자
template<typename F, typename... Types>
void howLong(F f, Types... args) {
    // 시간 측정
    f(args...);
    // 시간 측정 후 출력
}

int main() {
    foo(0);                 // foo
    howLong(foo, 0);        // foo

    goo(0);                 // goo
                            // 포인터로 암시적 변환 되어 goo((char*)0) 호출이 된다.
    howLong(goo, 0);        // error C2664: 'void (char *)': 인수 1을(를) 'int'에서 'char *'(으)로 변환할 수 없습니다.
                            // Types... 가 int 가 된다.
                            // 결국 goo((int)0); goo(int) 를 찾을 수 없어 error.
    howLong(goo, nullptr);  // Types... 가 nullptr_t 가 되어 char* 로 암시적 변환 된다.
                            // 결국 goo(nullptr);

    int n = 0;
    std::cin >> n;
    howLong(hoo, n, 2);     // hoo
    howLong(koo);           // koo
}
```