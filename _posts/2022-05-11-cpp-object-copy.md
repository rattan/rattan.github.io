---
published: true
title: "객체 복사"
date: 2022-05-11 21:16:10 +9:00
last_modified_at: 2022-05-11 21:16:11 +9:00
categories: cpp
---
포인터 멤버를 가진 클래스가 있다면 반드시 복사 생성자를 만들어야 한다.<p>
그렇지 않다면 컴파일러가 만드는 디폴트 생성자가 얕은 복사를 한다.
```cpp
#include <iostream>

class Cat {
    char* name;
    int age;
public:
    Cat(const char* n, int a) : age(a) {
        name = new char[strlen(n) + 1];
        strcpy(name, n);
    }

    ~Cat() {
        delete[] name;
    }

    // 자기 자신의 타입 하나를 인자로 가지는 생성자를 컴파일러가 만들어 준다
    // 흔히 복사 생성자 라고 한다.
    Cat(const Cat& c) {
        name = c.name;
        age = c.age;
    }
};

int main() {
    Cat c1("NABI", 2);
    Cat c2 = c1;    // runtime error. 얕은 복사(shallow copy)
    Cat c3(c1);     // 위 한줄은 이 표현과 유사하다.
}
```
---
깊은 복사
```cpp
#include <iostream>

class Cat {
    char* name;
    int age;
public:
    Cat(const char* n, int a) : age(a) {
        name = new char[strlen(n) + 1];
        strcpy(name, n);
    }

    ~Cat() {
        delete[] name;
    }

    Cat(const Cat& c) {
        age = c.age;                            // 깊은 복사(deep copy) 를 이용한 복사 생성자
                                                // 포인터가 아닌 멤버는 그냥 복사
        name = new char[strlen(c.name) + 1];    // 포인터 멤버는 메모리 할당 후
        strcpy(name, c.name);                   // 메모리를 통째로 복사
    }
};

int main() {
    Cat c1("NABI", 2);
    Cat c2(c1);
}
```
---
참조계수 를 사용한 복사 생성자
1. 참조 계수는 동적 메모리 할당 해야 한다.
   - static 멤버 data 로 하면 안된다.
2. COW(Copy On Write)
   - 공유 자원의 상태가 변경될 때 자원을 분리해야 한다.
```cpp
#include <iostream>

class Cat {
    char *name;
    int age;
    int *ref;               // 참조계수 를 가르킬 변수
    // static int ref;      // name 이 달라도 ref 변수는 하나밖에 없다.
public:
    Cat(const char* n, int a) : age(a) {
        name = new char[strlen(n) + 1];
        strcpy(name, n);

        ref = new int(1);   // 한 개를 1 로 초기화
    }

    // 참조계수 방식의 소멸자
    ~Cat() {
        if (--(*ref) == 0) {
            delete []name;
            delete ref;
        }
    }

    // 참조계수 로 구현 한 복사 생성자
    Cat(const Cat& c) {
        name = c.name;  // 먼저 모든 멤버를 얕은 복사 한다.
        age = c.age;    // 포인터, 값 모두 얕은 복사
        ref = c.ref;    // 참조계수 도 복사

        ++(*ref);       // 참조계수 1 증가
    }
};

int main() {
    Cat c1("NABI", 2);
    Cat c2(c1);

    // c1.setName("AAA");   // c1, c2 의 자원은 이제 분리 되어야 한다.
                            // Copy On Write 개념.

    Cat c3("AA", 4);
    Cat c4(c3);
}
```
---
소유권 이전
```cpp
#include <iostream>

class Cat {
    char *name;
    int age;
public:
    Cat(const char* n, int a) : age(a) {
        name = new char[strlen(n) + 1];
        strcpy(name, n);
    }

    ~Cat() {
        delete[] name;
    }

    // 소유권 이전의 복사 생성자: 어렵고 중요한 개념!
    // move 생성자  라고 부르는 개념
    Cat(Cat& c) {   // const 가 들어가지 않는다.
        // 얕은 복사 후에
        name = c.name;
        age = c.age;

        // 원본을 0 으로
        c.name = nullptr;
        c.age = 0;
    }

    int getAge() {
        return age;
    }
};

int main() {
    Cat c1("NABI", 2);
    Cat c2(c1);

    std::cout << c1.getAge() << std::endl;  // 0
    std::cout << c2.getAge() << std::endl;  // 2
}
```
---
```cpp
#include <iostream>

class Cat {
    char* name;
    int age;
public:
    Cat(const char* n, int a) : age(a) {
        name = new char[strlen(n) + 1];
        strcpy_s(name,strlen(n), n);
    }

    ~Cat() {
        delete[] name;
    }

    // c++98: 생성자, 복사 생성자
    // c++11: 생성자, 복사 생성자, Move 생성자

    // 복사 생성자: 예전 방식과 모양 동일
    Cat(const Cat& c) {
        // 깊은 복사 나 참조계수 기법으로 구현
        std::cout << "복사 생성자" << std::endl;
    }

    // Move 생성자: rvalue reference 를 사용하기도 한다.
    Cat(Cat&& c) {
        std::cout << "Move 생성자" << std::endl;

        // 소유권 이전 으로 구현
        name = c.name;      // 얕은복사 이후
        age = c.age;
        c.name = nullptr;   // 원본을 0 으로
        c.age = 0;
    }
};

template<typename T> void swap(T& a, T& b) {
    // 일반적인 swap 알고리즘
    T tmp = a;  // 복사 생성자 호출
    a = b;      // 대입 연산자 호출
    b = tmp;    // 대입 연산자 호출
}

template<typename T> T&& move(T& a) {
    return static_cast<T&&>(a); // move 생성자 호출을 위해 rvalue reference 로 캐스팅.
}

template<typename T> void swap2(T& a, T & b) {
    T tmp = move(a);    // static_cast<T&&>(a);
                        // std::move(a);
    a = b;              // 대입 연산자 호출
    b = tmp;            // 대입 연산자 호출
}

int main() {
    Cat c1("NABI", 2);
    Cat c2("AA", 2);
    swap2(c1, c2);      // error. 대입 연산자를 정의 해 주어야 한다.
}
```
---
```cpp
#include <iostream>

class Cat {
public:
    Cat() {
        std::cout << "생성자" << std::endl;
    }
    ~Cat() {
        std::cout << "소멸자" << std::endl;
    }

    Cat(const Cat&) {
        std::cout << "복사 생성자" << std::endl;
    }
    Cat& operator=(const Cat&) {
        std::cout << "복사 대입 연산자" << std::endl;
        return *this;
    }
    
    Cat(Cat&&) {
        std::cout << "Move 생성자" << std::endl;
    }
    Cat& operator=(Cat&&) {
        std::cout << "Move 대입 연산자" << std::endl;
        return *this;
    }
};

template<typename T> void swap(T& a, T& b) {
    T tmp = a;  // 복사 생성자
    a = b;      // 복사 대입 연산자
    b = tmp;    // 복사 대입 연산자
}               // 소멸자 (tmp)

template<typename T> void swap2(T& a, T& b) {
    T tmp = std::move(a);   // Move 생성자
    a = std::move(b);       // Move 대입 연산자
    b = std::move(tmp);     // Move 대입 연산자
}                           // 소멸자 (tmp)

int main() {
    Cat c1, c2;     // 생성자
                    // 생성자
    swap(c1, c2);
    swap2(c1, c2);
}                   // 소멸자
                    // 소멸자
```
---
복사 금지
 - 복사(생성, 대입) 을 막고싶을때
 - private 복사 생성자
 - 복사생성자, 복사대입 연산자를 private 으로 선언만 한다.
```cpp
#include <iostream>

class Cat {
    char* name;
    int age;
public:
    Cat(const char* n, int a) : age(a) {
        name = new char[strlen(n) + 1];
        strcpy_s(name, strlen(n), n);
    }
    ~Cat() {
        delete[] name;
    }
private:                // private 복사 생성자: 복사를 금지 하고 싶을때 사용하는 기술
    //Cat(const Cat&);  // 핵심. 선언만 한다.
    //Cat& operator=(const Cat&);
public:
    // c++11 은 위 개념을 문법화 할 수 있다.
    Cat(const Cat&) = delete;   // 복사 생성자를 지워달라.
    Cat& operator=(const Cat&) = delete;
};

int main() {
    Cat c1("NABI", 2);

    Cat c2 = c1;    // error C2280: 'Cat::Cat(const Cat &)': 삭제된 함수를 참조하려고 합니다.
                    // 복사 금지
}
```
[C2280 함수 "Cat::Cat(const Cat &)" (선언됨 줄 19)을(를) 참조할 수 없습니다. 삭제된 함수입니다.]: https://docs.microsoft.com/ko-kr/cpp/error-messages/compiler-errors-1/compiler-error-c2280
> error code
>- [C2280 함수 "Cat::Cat(const Cat &)" (선언됨 줄 19)을(를) 참조할 수 없습니다. 삭제된 함수입니다.][]
---
```cpp
#include <iostream>

class Cursor {
private:
    Cursor() {}
    static Cursor* instance;
public:
    static Cursor& getInstance() {
        if (instance == nullptr) {
            instance = new Cursor();
        }
        return *instance;
    }

    // 싱글톤은 복사와 대입을 제거 해야 한다.
    Cursor(const Cursor&) = delete;
    Cursor& operator=(const Cursor&) = delete;
};
Cursor* Cursor::instance = nullptr;

int main() {
    Cursor& c1 = Cursor::getInstance();
    Cursor c2 = c1; // 복사 금지
}
```
---
객체를 복사하는 방법 정리
- 디폴트 복사 생성자
  - 얕은 복사
- 얕은 복사를 해결하는 4가지 기술
   1. 깊은 복사(Deep Copy)
   2. 참조 계수(reference counting)
   3. 소유권 이전: Move 생성자로 발전됨
      - 복사와 Move 를 동시에 지원 하도록 클래스를 설계 하자
   4. 복사 금지: delete function 개념으로 발전됨