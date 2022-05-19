---
title: "STL 알고리즘"
date: 2022-05-19 21:10:36 +9:00
last_modified_at: 2022-05-19 21:10:38 +9:00
categories: cpp
---
setp1. c 의 strchr() 함수
```cpp
#include <iostream>

char* xstrchr(char* s, char c) {
    while (*s != 0 && *s != c) {
        ++s;
    }
    return *s == 0 ? 0 : s;
}

int main() {
    char s[] = "abcdefgh";
    char* p = xstrchr(s, 'c');

    if (p == 0) {
        std::cout << "찾을 수 없습니다." << std::endl;
    } else {
        std::cout << *p << std::endl;   // c
    }
}
```
---
일반화(Generic) 프로그래밍
 - 하나의 함수/클래스를 최대한 활용 범위가 넓도록 만들어 가는것
 - 하나의 함수로 다양한 경우를 처리하도록 하자

step2. 검색 범위의 일반화: 부분 문자열 검색이 가능하도록 하자
```cpp
#include <iostream>

// [first, last) 사이의 부분 문자열 검색
char* xstrchr(char* first, char* last, char c) {
    while (first != last && *first != c) {
        ++first;
    }
    return first == last ? 0 : first;
}

int main() {
    char s[] = "abcdefgh";
    char* p = xstrchr(s, s + 4, 'e');

    if (p == 0) {
        std::cout << "찾을 수 없습니다." << std::endl; // 찾을 수 없습니다.
    } else {
        std::cout << *p << std::endl;
    }
}
```
---
step3. 검색 대상 타입의 일반화: 검색 대상이 문자열로 한정될 필요가 없다
 - template 사용
```cpp
#include <iostream>

template<typename T>
T* xstrchr(T* first, T* last, T c) {
    while (first != last && *first != c) {
        ++first;
    }
    return first == last ? 0 : first;
}

int main() {
    int a[] = { 1,2,3,4,5 };
    int* p2 = xstrchr(a, a + 4, 2);
    if (p2 == 0) {
        std::cout << "찾을 수 없습니다." << std::endl;
    }
    else {
        std::cout << *p2 << std::endl;                  // 2
    }
}
```
---
step4. 구간의 타입과 검색 대상 타입의 분리
 - 구간의 타입과 찾는 요소의 타입이 연관된다
   - double 배열에서 int 요소를 찾을수 있어야 한다
 - T* 라고 하면 진짜 포인터만 사용할 수 있다.
   - 인자를 스마트포인터나 반복자를 사용할수 있게 해야 한다.
```cpp
#include <iostream>

template<typename T,  typename V>
T xstrchr(T first, T last, V c) {
    while (first != last && *first != c) {
        ++first;
    }
    return first == last ? 0 : first;
}

int main() {
    double d[] = { 1,2,3,4,5 };
    double* p = xstrchr(d, d + 4, 2);

    if (p == 0) {
        std::cout << "찾을 수 없습니다." << std::endl;
    } else {
        std::cout << *p << std::endl;   // 2
    }
}
```
---
step5. 실패의 전달
 - last 는 검색에 포함되지 않는다: past the end 라고 부른다
 - 실패할 경우, last 를 리턴 한다.
```cpp
#include <iostream>

template<typename T,  typename V>
T xstrchr(T first, T last, V c) {
    while (first != last && *first != c) {
        ++first;
    }
    return first;
}

int main() {
    double d[] = { 1,2,3,4,5 };
    double* p = xstrchr(d, d + 4, 7);   // 검색에 실패해서 last(d + 4) 를 리턴 받았다.

    if (p == d + 4) {
        std::cout << "찾을 수 없습니다." << std::endl; // 찾을 수 없습니다.
    } else {
        std::cout << *p << std::endl;
    }
}
```