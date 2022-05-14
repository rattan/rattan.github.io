---
published: true
title: "스마트 포인터"
date: 2022-05-15 00:05:40 +9:00
last_modified_at: 2022-05-15 00:05:42 +9:00
categories: cpp
---
스마트 포인터
 - 임의의 객체가 다른 타입의 포인터 처럼 사용되는 것
 - \* 연산자와 -> 연산자를 재정의 해서 포인터 처럼 보이게 한다
   - \* 연산자는 참조 리턴을 해야 한다.
장점
 - 진짜 포인터가 아니라 객체 이다.
 - 생성/복사/대입/소멸의 모든 과정을 사용자가 제어할 수 있다.
 - 대표적인 활용이 소멸자에서의 자동 삭제 기능
   - garbage collector 와 유사한 기능
```cpp
#include <iostream>

class Car {
public:
    void Go() {
        std::cout << "Car Go" << std::endl;
    }
    ~Car() {
        std::cout << "~Car" << std::endl;
    }
};

// 아래 클래스가 핵심이다.
class Ptr {
    Car* obj;
public:
    Ptr(Car* p = nullptr) :obj(p) {}
    
    ~Ptr() {
        delete obj;
    }
    
    Car* operator->() {
        return obj;
    }

    // 반드시 참조리턴 해야 한다.
    // Car 리턴 은 임시객체 이다: 다른 Car 객체가 만들어 진다.
    Car& operator*() {
        return *obj;
    }
};

int main() {
    Ptr p = new Car;    // Ptr p(new Car) 로 생각해 보자
    p->Go();            // (p.operator->()) G0() 의 모양 이지만
                        // (p.operator->())->Go() 로 해석해 준다.
    (*p).Go();          // 진짜 포인터 처럼 보이려면 *p.Go() 도 되어야 한다.
}
```
1. template 으로 만들어야 한다.
2. 얕은 복사 문제를 해결해야 한다.
   - 깊은 복사: 스마트 포인터 만들때는 거의 사용하지 않는다
   - 참조 계수: 스마트 포인터 만들때 가장 널리 사용되는 방식
   - 소유권 이전: ???
   - 복사 금지: 요즘 뜨고 있는 방식
```cpp
template<typename T>
class Ptr {
    T* obj;
public:
    Ptr(T* p = nullptr) : obj(p) {}
    ~Ptr() {
        delete obj;
    }
    T* operator->() {
        return obj;
    }
    T& operator*() {
        return *obj;
    }
};

int main() {
    Ptr<int> p1 = new int;
    Ptr<int> p2 = p1;
}
```
참조 계수 방식
```cpp
template<typename T>
class Ptr {
    T* obj;
    int* ref;
public:
    Ptr(T* p = nullptr) : obj(p) {
        ref = new int(1);
    }
    // 참조 계수 방식의 복사 생성자
    Ptr(const Ptr& p) {
        // 얕은복사 이후
        obj = p.obj;
        ref = p.ref;
        // 참조 계수 증가
        ++(*ref);
    }
    ~Ptr() {
        if (--(*ref) == 0) {
            delete obj;
            delete ref;
        }
    }
    T* operator->() {
        return obj;
    }
    T& operator*() {
        return *obj;
    }
};

int main() {
    Ptr<int> p1 = new int;
    Ptr<int> p2 = p1;
}
```
---
```cpp
class Animal {};
class Dog : public Animal {};

template<typename T>
class Ptr {
    T* obj;
    int* ref;
public:
    // 변환 생성자를 막기 위해 explicit 로 선언 한다.
    explicit Ptr(T* p = nullptr) : obj(p) {
        ref = new int(1);
    }
    // 일반 복사생성자
    Ptr(const Ptr& p) {
        obj = p.obj;
        ref = p.ref;
        ++(*ref);
    }

    // 일반화 된 복사 생성자
    // 부모 포인터로 자식 포인터를 가르키게 해야 한다.
    template<typename U>
    Ptr(const Ptr<U>& p) {
        obj = p.obj;
        ref = p.ref;
        ++(*ref);
    }
    template<typename> friend class Ptr;

    // 그 외에 대입연산(=), ==, != 연산자가 template 버전으로 있어야 한다.
    ~Ptr() {
        if (--(*ref) == 0) {
            delete obj;
            delete ref;
        }
    }
    T* operator->() {
        return obj;
    }
    T& operator*() {
        return *obj;
    }
};

int main() {
    Ptr<Dog> p1(new Dog);   // explicit 생성자이기 때문에 () 초기화만 가능하다.
    Ptr<Animal> p2 = p1;    // 다형성: 부모 포인터로 자식을 가르킬 수 있어야 한다.
}
```
이미 c++ 표준에 참조계수 스마트 포인터가 있다
```cpp
#include <memory>

int main() {
    std::shared_ptr<int> p1(new int);
    std::shared_ptr<int> p2 = p1;   // 이 순간 참조계수가 증가 한다.
}
```
---
참조 계수를 객체 안에 포함 시키자
1. 객체 생성 시 참조계수 증가
2. 객체 소멸 시 참조계수 감소
3. 객체 사용 후 참조계수 감소
```cpp
#include <iostream>

class Car {
    // Car 의 고유 멤버들
    int mCount; // 참조 계수
public:
    Car() : mCount(0) {}
    ~Car() {
        std::cout << "~Car" << std::endl;
    }

    void incStrong() {
        ++mCount;
    }
    void decStrong() {
        if (--mCount == 0) {
            delete this;
        }
    }
};

int main() {
    Car* p1 = new Car;
    p1->incStrong();
    Car* p2 = p1;
    p2->incStrong();
    p2->decStrong();
    p1->decStrong();
}   // ~Car
```
---
객체 안의 참조 계수를 자동으로 관리 해 주는 스마트 포인터
```cpp
#include <iostream>

class Car {
    // Car 의 고유 멤버들
    int mCount; // 참조 계수
public:
    Car() : mCount(0) {}
    ~Car() {
        std::cout << "~Car" << std::endl;
    }

    void incStrong() {
        ++mCount;
    }
    void decStrong() {
        if (--mCount == 0) {
            delete this;
        }
    }
};

template<typename T>
class sp {
    T* obj;
public:
    sp(T* p = nullptr) :obj(p) {
        if (obj) {
            obj->incStrong();
        }
    }
    sp(const sp& p) :obj(p.obj) {
        if (obj) {
            obj->incStrong();
        }
    }
    ~sp() {
        if (obj) {
            obj->decStrong();
        }
    }

    // 스마트 포인터의 기본 * 와 ->
    T* operator->() {
        return obj;
    }
    T& operator*() {
        return *obj;
    }
};

// sp 를 사용하려면 반드시 incStrong() 과 decStrong() 이 있어야 한다.
// sp 를 사용하기 위해 _RefBase_ 의 파생 클래스로 만들자
class Truck/* :public RefBase*/ {
public:
    ~Truck() {
        std::cout << "~Truck" << std::endl;
    }
};

int main() {
    sp<Car> p1 = new Car;
    sp<Car> p2 = p1;

    sp<Truck> p3 = new Truck;

}   // ~Car
```
---
객체 관리의 어려움
작성중 ~2022-05-15 00:05:36 +9:00