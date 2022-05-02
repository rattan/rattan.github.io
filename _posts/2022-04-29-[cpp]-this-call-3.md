---
title: "this call 3"
date: 2022-05-02 19:55:02
categories: c++
---
NULL 객체의 함수 호출 문제
```cpp
class Test {
    int data;
public:
    void f1() {
        std::cout<<"f1"<<std::endl;
    }
    int f1() {
        std::cout<<"f2"<<std::endl;
        retrun 0;
    }
    int f1() {
        std::cout<<"f1"<<std::endl;
        return data;                // this->data;
    }

    int call_f3() {
        return this ? f3() : 0;     // NULL 객체에 대해 함수를 호출 해도 죽지않게 하기 위한 함수
    }

    virtual void f4() {}
};

int main() {
    Test *p = nullptr;              // 메모리 할당에 실패해 nullptr 이 나왔다고 가정

    p->f1();                        // f1(p), f1(nullptr) : 어떻게 될까?
    p->f2();                        // ok.
    p->f3();                        // error. this이(가) nullptr였습니다.
    p->call_f3();                   // ok.
    p->f4();                        // nullptr 번지의 가상함수 테이블을 찾으려 한다.
                                    // runtime error.
}
```