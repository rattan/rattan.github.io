---
title: "begin end"
date: 
last_modified_at: 
categories: cpp
---
```cpp
#include <iostream>
#include <vector>
#include <type_traits>

// 컨테이너의 모든 요소를 화면에 출력하는 함수
template<typename T>
void showImp(T& c, std::false_type) {
    typename T::iterator p = c.begin();

    while (p != c.end()) {
        std::cout << *p << " ";
        ++p;
    }
    std::cout << std::endl;
}

// 배열일 때
template<typename T>
void showImp(T& c, std::true_type) {
    auto p = c; // 배열의 이름이 시작주소 이다.

    while (p != c + std::extent<T, 0>::value) {
        std::cout << *p << " ";
        ++p;
    }
    std::cout << std::endl;
}

template<typename T>
void show(T& c) {
    showImp(c, std::is_array<T>());
}

int main() {
    int x[10] = { 1,2,3,4,5,6,7,8,9,10 };
    std::vector<int> v(x, x + 10);

    show(v);    // 1 2 3 4 5 6 7 8 9 10
    show(x);    // 1 2 3 4 5 6 7 8 9 10
}
```
---
기존의 스타일
1. 배열에는 begin(), end() 가 없다.
2. 배열버전을 따로 만들어야 한다
3. show 에서는 is_array<> 를 사용해서 함수 오버로딩으로 해결

c++11 새로운 스타일
```cpp
// 컨테이너 버전
template<typename T>
auto begin(T& c) {
    return c.begin();
}
template<typename T>
auto end(T& c) {
    return c.end();
}

// 배열 버전
template<typename T, int N>
auto begin(T(&arr)[N]) {
    return arr;
}
template<typename T, int N>
auto end(T(&arr)[N]) {
    return arr + N;
}
```
멤버함수 begin()/end() 대신 일반함수 begin()/end() 를 사용하자.
 - 하나의 함수에서 객체형 컨테이너와 일반 배열을 모두 처리할수 있게 된다.
 - is_array<T>() 등의 함수오버로딩이 필요없어진다.
```cpp
#include <iostream>
#include <vector>

template<typename T>
void show(T& c) {
    auto p = std::begin(c);
    while (p != std::end(c)) {
        std::cout << *p << " ";
        ++p;
    }
    std::cout << std::endl;
}

int main() {
    int x[10] = { 1,2,3,4,5,6,7,8,9,10 };
    std::vector<int> v(x, x + 10);

    show(v);    // 1 2 3 4 5 6 7 8 9 10
    show(x);    // 1 2 3 4 5 6 7 8 9 10
}
```
