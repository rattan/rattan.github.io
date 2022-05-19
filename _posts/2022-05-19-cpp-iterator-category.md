---
title: "iterator category"
date: 2022-05-19 22:01:34 +9:00
last_modified_at: 2022-05-19 22:01:36 +9:00
categories: cpp
---
```cpp
#include <iostream>
#include <list>
#include <algorithm>    // find() 와 같은 함수가 여기에 있다.

int main() {
    std::list<int> s;
    s.push_back(10);
    s.push_back(20);

    // stl 함수들
    // - 다양한 타입에 적용되는 일반화 함수들
    // - find() 를 가지고 배열, list, vector 등에서 사용할 수 있다.
    std::sort(s.begin(), s.end());
}
```
---
우리가 만든 slist 를 생각해 보자.
```cpp
int main() {
    slist<int> s;
    s.push_front(10);
    s.push_front(10);

    slist<int>::iterator p = s.begin();
    ++p;    // ok.
    --p;    // ??
            // 문법적으로는 문제 없지만 싱글 리스트라는 자료구조 특성상 만들 수 없다.
}
```
---
반복자 카테고리
1. 입력 반복자
   - =*p
   - ++
2. 출력 반복자
   - *p=
   - ++
3. 전진형 반복자: 싱글 리스트
   - 입출력
   - ++
4. 양방향 반복자: 더블 리스트
   - 입출력
   - ++
   - \--
5. 임의 접근 반복자: 연속된 메모리
   - 입출력
   - ++
   - \--
   - \+
   - \-
   - []
```cpp
#include <vector>
#include <list>
#include <algorithm>

int main() {
    int x[10] = { 1,2,3,4,5,6,7,8,9,10 };
    int* p = std::find(x, x + 10, 5);   // 1, 2번째 인자는 반복자 이다.
                                        // 최소 요구 조건: 입력 반복자
    std::reverse(x, x + 10);            // 최소 조건: 양방향 반복자
    std::sort(x, x + 10);               // 최소 조건: 임의 접근 반복자

    // slist<int> s;                    // 우리가 만든 slist
    // std::reverse(s.begin(), s.end());// 가능할까?

    std::list<int> s2;
    std::sort(s2.begin(), s2.end());    // error C2676: 이항 '-': 'const std::_List_unchecked_iterator<std::_List_val<std::_List_simple_types<_Ty>>>'이(가) 이 연산자를 정의하지 않거나 미리 정의된 연산자에 허용되는 형식으로의 변환을 정의하지 않습니다.
                                        // list 는 임의접근 반복자가 없어서 sort() 를 사용 할 수 없다.
    s2.sort();                          // 일반화 함수 sort() 를 사용할수 없기때문에 멤버함수 sort() 가 있다.
                                        // 임의접근해야하는 quick sort 가 아닌 다른 알고리즘 사용

    std::vector<int> v;
    v.sort();                           // vector 에는 sort() 멤버함수가 없다.
                                        // vector의 반복자는 임의접근 반복자 이므로 일반화 함수 sort() 사용가능 하므로 멤버 sort() 가 필요 없다.
}
```
---
STL 에서는 반복자의 5개 개념을 타입화 한다.<p>
empty class
 - 아무 멤버도 없는 구조체(클래스)
 - 멤버가 없어도 타입은 타입이다.
 - 대부분, 함수 오버로딩이나 템플릿 인자로 활용된다.

모든 반복자의 설계자는 자신이 어떤 종류인지 외부에 알려야 한다.
```cpp
struct input_iterator_tag {};
struct output_iterator_tag {};
struct forward_iterator_tag : input_iterator_tag {};
struct bidirectional_iterator_tag : forward_iterator_tag {};
struct random_access_iterator_tag : bidirectional_iterator_tag {};

template<typename T>
class vector_iterator {
public:
    typedef random_access_iterator_tag iterator_category;
    // ...
};

template<typename T>
class list_iterator {
    typedef bidirectional_iterator_tag iterator_category;
    // ...
};
```