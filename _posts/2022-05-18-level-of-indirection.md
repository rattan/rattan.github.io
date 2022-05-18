---
title: "간접층의 원리"
date: 2022-05-18 21:42:51 +9:00
last_modified_at: 2022-05-18 21:42:52 +9:00
categories: cpp
---
간접층의 원리: Level of Indirection
 - 아무리 어려운 문제점도 중간층(기존 요소 사이에 새로운요소) 를 도입하면 해결 할 수 있다.
 - 기존 요소를 대신한다는 의미로 Proxy Pattern 이라고도 불리는 디자인 기법
```cpp
struct VectorSize {
    int size;
    VectorSize(int n) : size(n) {}  // int => VectorSize 로 암시적 변환
};

class Vector {
public:
    // Vector(int sz) {}
    Vector(VectorSize sz) {         // VectorSize => Vector 로 암시적 변환
        int size = sz.size;
    }
};

void foo(Vector v) {}

int main() {
    Vector v(10);
    foo(v);     // ok.
    foo(20);    // error C2664: 'void foo(Vector)': 인수 1을(를) 'int'에서 'Vector'(으)로 변환할 수 없습니다.
                // int => Vector 즉, 변환생성자가 있으면 ok.
}
```
c++ programming language 4/e 책 13장에 String 전체 코드가 있다.
 - Bjarne Stroustrup
```cpp
#include <iostream>

class String {
    char* buf;
    int* ref;   // 참조 계수
public:
    String(const char* s) {
        buf = new char[strlen(s) + 1];
        strcpy_s(buf, strlen(s) + 1, s);
        ref = new int(1);
    }

    String(const String& s) {
        buf = s.buf;
        ref = s.ref;
        ++(*ref);
    }

    ~String() {
        if (--(*ref) == 0) {
            delete[]buf;
            delete ref;
        }
    }

    void print() const {
        std::cout << buf << std::endl;
    }

    // 문제를 해결하기 위해 새로운 요소를 도입한다.
    // char 를 대신하는 요소를 만들자
    class CharProxy {
        String& str;
        int index;
    public:
        CharProxy(String& s, int n) : str(s), index(n) {}

        // Proxy 는 char 로 변환 할 수 있어야 한다.
        operator char() {
            std::cout << "읽는 작업중. 복사본 필요 없다." << std::endl;
            return str.buf[index];
        }

        CharProxy& operator=(char c) {
            std::cout << "쓰는 작업 중. 복사본을 만들어야 한다." << std::endl;
        
            // 버퍼 복사본
            char* temp = new char[strlen(str.buf) + 1];
            strcpy_s(temp, strlen(str.buf) + 1, str.buf);

            // 참조계수 1 감소
            if (--(*str.ref) == 0) {
                delete[] str.buf;
                delete str.ref;
            }
            str.buf = temp;
            str.ref = new int(1);

            str.buf[index] = c;
            return *this;
        }
    };

    CharProxy operator[](int index) {
        std::cout << "operator[]" << std::endl;
        return CharProxy(*this, index);
    }

    // []연산자 재정의: 객체를 배열처럼 사용가능하게 한다.
    // s[0] = 'a' 처럼 [] 호출이 lvalue에 오게 하려면 참조리턴 해야 한다.
    // char& operator[](int index) {
    //     std::cout << "operator[]" << std::endl;
    //     return buf[index];
    // }
    // read 작업은 ok.
    // write 작업 시에는 자원을 분리해서 작업해야 한다.
    // Copy On Write.
};

int main() {
    String s1 = "hello";
    String s2 = s1;             // 참조 계수 방식의 복사

    char c = s1[0];             // operator[]
                                // 읽는 작업중. 복사본 필요 없다.
                                // 읽는 작업이므로 자원(문자열) 은 계속 공유 되어야 한다.
    std::cout << c << std::endl;// h

    s1[0] = 'x';                // operator[]
                                // 쓰는 작업중. 복사본을 만들어야 한다.
                                // 쓰는 작업이다. 자원을 분리하고 변경해야 한다.
                                // Copy On Write

    s1.print();                 // xello
    s2.print();                 // hello
}
```