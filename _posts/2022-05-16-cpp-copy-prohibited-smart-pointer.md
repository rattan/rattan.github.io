---
title: "복사금지 스마트 포인터"
date: 2022-05-16 19:03:23 +9:00
last_modified_at: 2022-05-16 19:03:25 +9:00
categories: cpp
---
복사금지: 요즘 뜨고 있는 방식<p>
단지 자원 관리 목적으로만 사용된다면, 참조계수 방식 보다는 복사 금지 방식의 스마트 포인터가 더 좋다.
 - 모든 함수가 inline 치환 된다.
 - 참조계수를 위한 어떠한 메모리도 필요없다.
 - 스마트 포인터 도입에 따른 오버헤드가 전혀 없다.
 - c++ 표준의 std::unique_ptr<> 클래스가 복사금지 방식
```cpp
template<typename T>
class Ptr {
    T* obj;
    
    // 단순히 자원관리 용도로만 사용할 때에는 참조계수도 오버헤드이다.
    // 자원관리 전용 스마트 포인터도 제공 하자: 복사금지 스마트 포인터
    Ptr(const Ptr&) = delete;
    Ptr& operator=(const Ptr&) = delete;
public:
    inline Ptr(T* p = nullptr) :obj(p) {}
    inline ~Ptr() {
        delete obj;
    }
    inline T* operator->() {
        return obj;
    }
    inline T& operator*() {
        return *obj;
    }
};

int main() {
    Ptr<int> p1 = new int;
    Ptr<int> p2 = p1;   // error C2280: 'Ptr<int>::Ptr(const Ptr<int> &)': 삭제된 함수를 참조하려고 합니다.
                        // 복사 금지
}
```
---
메모리 해지 전략을 담은 함수 객체들
```cpp
#include <iostream>

struct Freer {
    inline void operator()(void* p) const {
        std::cout << "free 사용" << std::endl;
        free(p);
    }
};

template<typename T>
struct DefaultDelete {
    // void* 를 delete 하면, 소멸자가 호출되지 않는다.
    // T 타입을 받아 delete.
    // inline void operator()(void*p) const
    inline void operator()(T* p) const {
        std::cout << "delete 사용" << std::endl;
        delete p;
    }
};

template<typename T, typename D = DefaultDelete<T>>
class UniquePtr {
    T* obj;
    UniquePtr(const UniquePtr&) = delete;
    UniquePtr& operator=(const UniquePtr&) = delete;
public:
    inline explicit UniquePtr(T* p = nullptr) :obj(p) {}
    inline ~UniquePtr() {
        // D d;     // 함수 객체 생성
        // d(obj);  // 함수 객체 이므로 함수처럼 사용
        D()(obj);
    }
    inline T* operator->() {
        return obj;
    }
    inline T& operator*() {
        return *obj;
    }
};

int main() {
    UniquePtr<int, DefaultDelete<int>> p1(new int);
    UniquePtr<int, Freer> p2((int*)malloc(100));
    UniquePtr<int> p3(new int);
}   // delete 사용
    // free 사용
    //delete 사용
```
---
```cpp
#include <iostream>

struct Freer {
    inline void operator()(void* p) const {
        std::cout << "free 사용" << std::endl;
        free(p);
    }
};

template<typename T>
struct DefaultDelete {
    inline void operator()(T* p) const {
        std::cout << "delete 사용" << std::endl;
        delete p;
    }
};

// 배열 delete 를 위한 부분전문화 버전
template<typename T>
struct DefaultDelete<T[]> {
    inline void operator()(T* p) const {
        std::cout << "delete[] 사용" << std::endl;
        delete[]p;
    }
};

template<typename T, typename D = DefaultDelete<T>>
class UniquePtr {
    T* obj;
    UniquePtr(const UniquePtr&) = delete;
    UniquePtr& operator=(const UniquePtr&) = delete;
public:
    inline explicit UniquePtr(T* p = nullptr) :obj(p) {}
    inline ~UniquePtr() {
        D()(obj);
    }
    inline T* operator->() {
        return obj;
    }
    inline T& operator*() {
        return *obj;
    }
};

// 배열일때 를 위한 부분 전문화
// 부분 전문화 버전에서는 디폴트 인자를 표시하지 않게 된다.
// primary 버전의 디폴트 값을 사용하게 된다.
template<typename T, typename D>
class UniquePtr<T[], D> {
    T* obj;
    UniquePtr(const UniquePtr&) = delete;
    UniquePtr& operator=(const UniquePtr&) = delete;
public:
    inline explicit UniquePtr(T* p = nullptr) :obj(p) {}
    inline ~UniquePtr() {
        D()(obj);
    }

    // 배열 버전에서는 [] 연산사도 제공하면 좋다.
    inline T& operator[](int index) {   // 임시객체 생성을 막기 위해 참조 리턴
        return obj[index];
    }

    // 아래 2개도 있어야 하지만 c++ 표준의 배열버전 에서는 제거되어 있다.
    // inline T* operator->() { return obj; }
    // inline T& operator*() { return *obj; }
};

int main() {
    UniquePtr<int> p1(new int);
    UniquePtr<int[]> p2(new int[10]);   // 이 코드는 primary 버전에서 int[]* obj 로 생성된다
                                        // int[]* 는 잘못된 표현 이므로 부분전문화 버전이 필요하다.
    p2[0] = 10; // ok.
    p1[0] = 10; // error C2676: 이항 '[': 'UniquePtr<int,DefaultDelete<T>>'이(가) 이 연산자를 정의하지 않거나 미리 정의된 연산자에 허용되는 형식으로의 변환을 정의하지 않습니다.
                // primary 버전에서 [] 연산자가 정의되지도 않았고
                // p1 은 UniquePtr<int> 타입이지 int* 가 아니다.
    *p2 = 10;   // *연산자가 정의되어 있지 않다.
                // [] 연산자가 있으므로, *로 계산하지 않아도 된다.
    *p1 = 10;   // ok.

}
```
[UniquePtr.h]: https://android.googlesource.com/platform/system/chre/+/refs/heads/master/util/include/chre/util/unique_ptr.h
>[UniquePtr.h][]
---
복사금지 스마트 포인터
 - 오픈소스: UniquePtr<>
   - 안드로이드 프레임워크
   - 타이젠
 - c++11 표준: unique_ptr<>
   - 복사금지
   - 소유권 이전(move) 지원
```cpp
#include<memory>

int main() {
    std::unique_ptr<int> p1(new int);
    std::unique_ptr<int[]> p2(new int[10]);
    std::unique_ptr<int> p3 = p1;               // error C2280: 'std::unique_ptr<int,std::default_delete<int>>::unique_ptr(const std::unique_ptr<int,std::default_delete<int>> &)': 삭제된 함수를 참조하려고 합니다.
                                                // 복사 금지
    std::unique_ptr<int> p4 = std::move(p1);    // ok.
}
```