---
published: true
title: "thin template"
date: 2022-05-08 21:41:44 +9:00
last_modified_at: 2022-05-08 21:41:46 +9:00
categories: cpp
---
```cpp
template<typename T>
class vector {
    T buffer[100];
    int _size;
public:
    void push_front(T a) {}
    T front() { return buffer[0]; }
    int size() { return _size; }
    bool empty() { return _size == 0; }
};

int main() {
    vector<int> v1;
    vector<char> v2;
    vector<double> v3;
    // vector 의 멤버함수는 몇개 일까?
    // 함수4개 * 3개 타입: 생성되는 함수 12개
}
```
---
T 를 사용하지 않는 모든 멤버함수는 부모 클래스로 만든다.
```cpp
class vectorImpl {
    int _size;
public:
    int size() { return _size; }
    bool empty() { return _size == 0; }
};

template<typename T>
class vector : public vectorImpl {
    T buffer[100];
public:
    void push_front(T a) {}
    T front() { return buffer[0]; }
};

int main() {
    vector<int> v1;
    vector<char> v2;
    vector<double> v3;
    // 부모 함수 2개 + 자식 함수 2개 * 3개 타입: 8개 함수 생성
}
```
---
Thin Template
 - T 가 될 타입을 부모 클래스에 void* 로 만든다
 - 자식 템플릿을 만들어 캐스팅만 책임지도록 한다
```cpp
class vectorImpl {          // c 에서 배우는 void* 기반 컨테이너
    void* buffer[100];
    int _size;
public:
    int size() { return _size; }
    bool empty() { return _size == 0; }
    void push_front(void* a) {}
    void* front() { return buffer[0]; }
};

template<typename T>
class vector : private vectorImpl {
public:
    inline void push_front(T a) { vectorImpl::push_front((void*)a); }
    inline T front() { return static_cast<T>(vectorImpl::front()); }
    inline int size() { return vectorImpl::size(); }
    inline bool empty() { return vectorImpl::empty(); }
};

int main() {
    vector<int> v1;
    vector<char> v2;
    vector<double> v3;
}
```