---
title: "override"
date: 2022-05-21 18:45:40 +9:00
last_modified_at: 2022-05-21 18:45:41 +9:00
categories: cpp
---
```cpp
class Base {
public:
    virtual void foo(int) {}
    virtual void goo() const {}
    void hoo() {}
};

class Derived: public Base {
public:
    // 가상함수를 재정의 하고 싶을때 실수하기 쉬운것들
    // 정상 컴파일이 된다.
    void foo(int) {}            // ok. virtual 생략 가능.
    virtual void foo(double) {} // 파라미터 타입이 다르다.
    virtual void goo() {}       // 상수 함수가 아니다.
    virtual void gooo() {}      // 함수 이름이 다르다.
    void hoo() {}               // 가상 함수가 아니다.
};

class Derived2: public Base {
public:
    void foo(int) override {}           // ok. virtual 생략 가능.
    virtual void foo(double) override {}// error. 파라미터 타입이 다르다.
    virtual void goo() override {}      // error. 상수 함수가 아니다.
    virtual gooo() override {}          // error. 함수 이름이 다르다.
    void hoo override {}                // error. 가상함수가 아니다.
    virtual void foo(int) final {}      // Derived 의 자식클래스 부터는 foo 를 재정의 하지 못하게 한ㄷ
}

int main() {}
```