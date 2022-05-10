---
published: true
title: "단위전략"
date: 2022-05-08 22:46:03 +9:00
last_modified_at: 2022-05-08 22:46:05 +9:00
categories: cpp
---
단위 전략(policy base) 디자인 기술
 - 클래스 설계 시 정책을 담을 정책 클래스를 템플릿 인자로 교체 할 수 있게 디자인 하는 기술
 - 성능 저하 없이 정책을 변경 할 수 있다.
 - 모든 정책 클래스는 지켜야 하는 규칙이 있다.
   - lock()
   - unlock()
```cpp
#include <iostream>

template<typename T, typename ThreadModel = NoLock>
class list : public ThreadModel {
public:
    void push_front(const T& a) {
        ThreadModel::lock();
        // ...
        ThreadModel::unlock();
    }
};

// 컨테이너의 동기화를 위한 정책 클래스
class NoLock {
public:
    inline void lock() {}
    inline void unlock() {}
};

class MutexLock {
public:
    inline void lock() { std::cout << "Mutex lock" << std::endl; }
    inline void unlock() { std::cout << "Mutex unlock" << std::endl; }
};

list<int, MutexLock> s1;    // 전역 변수: 멀티 스레드 환경에서 안전하지 않다.
list<int, NoLock> s2;

int main() {
    s1.push_front(10);
}
```
---
```cpp
#include <iostream>

template<typename T>
class xallocator {
public:
    // 메모리 할당과 해지만
    T* allocate(int size) {
        return (T*)operator new(sizeof(T) * size);
    }
    void deallocate(T* p) {
        operator delete(p);
    }

    // 생성장 소멸자 호출을 별도로
    void construct(T* p) { new(p) T; }
    void destruct(T* p) { p->~T(); }
};

template<typename T, typename Alloc = xallocator<T>>
class vector {
    Alloc alloc;    // 단위 전략은 상속 또는 포함으로 사용해도 된다.
public:
    void resize(int size) {
        // 메모리가 부족해서 메모리 할당을 해야 한다.
        // malloc? new? pool? system call?
        T* p = alloc.allocate(size);

        // 메모리 해지
        alloc.deallocate(p);
    }
};

int main() {
    vector<int, xallocator<int>> v;
}
```
---
c++ 컨테이너에 메모리 할당 정책을 담당하는 클래스를 교체 할 수 있다.
```cpp
#include <iostream>
#include <vector>

template<class T>
class MyAlloc {
public:
    // 필요한 타입 선언
    typedef T value_type;
    typedef T* pointer;
    typedef const T* const_pointer;
    typedef T& reference;
    typedef const T& const_reference;
    typedef std::size_t size_type;
    typedef std::ptrdiff_t difference_type;

    // allocator type 을 U 로 rebind
    template<class U>
    struct rebind {
        typedef MyAlloc<U> other;
    };

    // 값의 주소를 반환하는 함수
    pointer address(reference value) const {
        return &value;
    }

    const_pointer address(const_reference value) const {
        return &value;
    }

    // 생성자와 소멸자
    // 아무 작업도 하지 않는다.
    MyAlloc() throw() {}
    MyAlloc(const MyAlloc&) throw() {}
    ~MyAlloc() throw() {}
    template<class U> MyAlloc(const MyAlloc<U>&) throw() {}

    // 최대 할당 가능 한 T 의 개수 리턴
    size_type max_size() const throw() {
        return std::numeric_limits<std::size_t>::max() / sizeof(T);
    }

    // 메모리 할당 하고 초기화는 하지 않는다.
    pointer allocate(size_type num, const void* = 0) {
        std::cerr << "allocate " << num << " element(s) of size " << sizeof(T) << std::endl;

        pointer ret = (pointer)(::operator new(num * sizeof(T)));

        std::cerr << "allocated at: " << (void*)ret << std::endl;
        
        return ret;
    }

    // p 가 가르키고 있는 항목을 value 매개변수로 초기화
    void construct(pointer p, const T& value) {
        // placement new 로 초기화
        new ((void*)p)T(value);
        std::cerr << "construct at: " << (void*)p << std::endl;
    }

    // p 가 가르키고 있는 항목 소멸 (소멸자만 호출)
    void destroy(pointer p) {
        // 소멸자 호출 하여 항목 정리(소멸)
        p->~T();
        std::cerr << "destroy at: " << (void*)p << std::endl;
    }

    // 메모리 해지
    void deallocate(pointer p, size_type num) {
        std::cerr << "deallocate " << num << " element(s) of size " << sizeof(T) << " at: " << (void*)p << std::endl;
        ::operator delete((void*)p);
    }
};

int main() {
    std::vector<int, MyAlloc<int>> v;   // allocate 1 element(s) of size 16
                                        // allocate at:  00000295B037C630
                                        // std::vector 의 메모리 할당
    v.push_back(10);                    // allocate 1 elemnet(s) of size 4
                                        // allocated at: 00000295B036E2F0
                                        // construct at: 00000295B036E2F0
                                        // push_back(10) 으로 삽입된 데이터의 메모리 할당 및 초기화
                                        // destroy at: 00000295B036E2F0
                                        // deallocate 1 element(s) of size 4 at: 00000295B036E2F0
                                        // deallocate 1 element(s) of size 16 at: 00000295B037C630
                                        // main 을 빠져나가면서
                                        // 1. 삽입된 데이터에 대함 소멸자 호출
                                        // 2. 삽입된 데이터의 메모리 해제
                                        // 3. std::vector 에 대한 메모리 해제
}
```
---
```cpp
#include <iostream>
#include <string>

// std::string 클래스의 모양은 아래와 유사하게 설계되어 있다.
template<typename T,    // UNICODE 고려
    typename traits,    // 단위 전략
    typename Alloc>     // 메모리 할당 정책
class basic_string {
    T* buff;
    Alloc alloc;
public:
    bool operator==(const basic_string& s) {
        return traits::cmp(buff, s.buff);   // 두 개의 문자열 비교
    }
};

typedef basic_string<char, std::char_traits<char>, std::allocator<char>> string;

// basic_string 이 사용 할 비교 단위 전략
// 단위 전략을 만들 때 일부 함수의 정책만 변경하려면
// 기존 단위 전략의 자식으로 만들면 된다.
struct my_traits : public std::char_traits<char> {
    static bool eq(char a, char b) {
        return toupper(a) == toupper(b);
    }
    static bool lt(char a, char b) {
        return toupper(a) < toupper(b);
    }
    static bool gt(char a, char b) {
        return toupper(a) > toupper(b);
    }
    static bool compare(const char* a, const char* b, int size) {
        std::cout << "cmp" << std::endl;
        return _memicmp(a, b, size);
    }
};

typedef std::basic_string<char, my_traits, std::allocator<char>> my_string;

int main() {
    my_string s1 = "ABCD";
    my_string s2 = "abcd";

    if (s1 == s2) {                             // cmp
        std::cout << "same" << std::endl;       // same
    }
    else {
        std::cout << "not smae" << std::endl;
    }
}
```