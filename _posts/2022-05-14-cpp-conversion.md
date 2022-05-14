---
published: true
title: "변환"
date: 2022-05-14 22:19:55 +9:00
last_modified_at: 2022-05-14 22:19:56 +9:00
categories: cpp
---
 - 변환 연산자
   - 객체(Point) => 다른 타입(int)
 - 변환 생성자
   - 다른 타입(int) => 객체(Point)
```cpp
class Point {
    int x, y;
public:
    Point() : x(0), y(0) {}
    Point(int a, int b) : x(a), y(b) {}

    // 변환 생성자: 인자가 한개인 생성자
    //              다른 타입이 객체로 암시적 형 변환 되게 한다.
    Point(int a) : x(a), y(0) {}

    // 변환 연산자: 객체를 다른 타입으로 암시적 형변환 되도록 한다.
    // 함수 이름에 리턴타입이 들어있으므로 리턴 타입을 표시하지 않는다.
    operator int() { return x; }
};

int main() {
    Point p1;
    Point p2(1, 2);
    
    int n = 10;
    n = p2; // Point => int 변환: p2.operator int() 가 필요
    p2 = n; // int => Point 변환: n.operator Point() 가 필요
            // 그런데 n 은 사용자 타입이 아니다.
}
```
---
생성자, 소멸자로 자원을 관리하면 편리하다
RAII(Resource Acquision Is Initialize) 라는 기술
```cpp
#include <iostream>

class OFile {
    FILE* file;
public:
    // explicit: 인자가 한개인 생성자가 암시적 변환을 일으키는 것을 막는다.
    //           단, 명시적 변환은 허용한다.
    explicit OFile(const char* name, const char* mode = "wt") {
        file = fopen(name, mode);
    }
    ~OFile() {
        fclose(file);
    }

    // OFile => FILE* 로의 변환을 허용 한다.
    operator FILE* () {
        return file;
    }
};

void foo(OFile f) {}

int main() {
    OFile f("file");
    foo(f);                             // ok. 당연하다.
    foo("hello");                       // error C2664: 'void foo(OFile)': 인수 1을(를) 'const char [6]'에서 'OFile'(으)로 변환할 수 없습니다.
                                        // explicit  로 인한 const char* => OFile 로 암시적 변환을 막는다.
    foo(static_cast<OFile>("hello"));   // ok. 명시적 변환
}
```
[C2664 "const char [6]"에서 "OFile"(으)로 변환하기 위한 적절한 생성자가 없습니다.]: https://docs.microsoft.com/ko-kr/cpp/error-messages/compiler-errors-2/compiler-error-c2664
> error code
>-[C2664 "const char [6]"에서 "OFile"(으)로 변환하기 위한 적절한 생성자가 없습니다.][]
---
 - explicit 생성자를 가지는 클래스는 = 로 초기화 할 수 없다.
 - 반드시 () 초기화만 사용해야 한다.
```cpp
#include <memory>

class Test {
    int data;
public:
    explicit Test(int n) : data(n) {}
};

void foo(std::shared_ptr<int> p) {}

int main() {
    // 아래 표현들의 차이점
    Test t1(5);     // 인자가 int 한개인 생성자 호출해서 객체 생성
    Test t2 = t1;   // 복사 생성자 호출
    Test t3 = 5;    // error C2440: '초기화 중': 'int'에서 'Test'(으)로 변환할 수 없습니다.
                    // 5 를 변환 생성자를 이용해서 Test 임시 객체 생성
                    // 복사 생성자를 호출 해서 임시객체를 t3 에 복사
                    // Test(int) 는 explicit 이므로 변환 생성자의 암시적 변환 막아서 error.

    std::shared_ptr<int> p1(new int);   // c++ 표준 스마트 포인터
    std::shared_ptr<int> p2 = new int;  // error C2440: '초기화 중': 'int *'에서 'std::shared_ptr<int>'(으)로 변환할 수 없습니다.

    foo(new int);                       // error C2664: 'void foo(std::shared_ptr<int>)': 인수 1을(를) 'int *'에서 'std::shared_ptr<int>'(으)로 변환할 수 없습니다.
                                        // int* => std::shared_ptr<int> 로 암시적 변환 되어야 한다.
                                        // std::shared_ptr<int> 는 explicit 생성자 이므로 error.
    foo(std::shared_ptr<int>(new int)); // ok. std::shared_ptr<int> 객체를 만들어서 전달.
}
```
---
```cpp
#include <iostream>

int main() {
    int n = 0;
    while (true) {
        std::cin >> n;          // 'a' 를 입력후 enter.
        if (std::cin.fail()) {
            std::cout << "다시 입력하세요" << std::endl;
            std::cin.clear();   // 내부적으로 관리되는 state
                                // 멤버를 reset 한 후 다시 입력받아야 한다.
            std::cin.ignore();  // 입력 버퍼를 비운다.
            continue;
        }
        break;
    }
    std::cout << n << std::endl;
}
```
---
```cpp
#include <iostream>

int main() {
    int n = 0;
    std::cin >> n;

    // 입력 성공을 조사하는 방법 1. fail() 멤버 함수
    if (std::cin.fail()) {}

    // 2. cin 자체를 if 로 조사: 원리가 무엇일까?
    if (std::cin) { // cin.operator void*() 가 있기때문에 가능 하다.
        std::cout << "성공" << std::endl;
    }
}
```
---
 - std::cin 은 istream 클래스 이다.
 - jerry schwarz 라는 개발자가 만들었다.
```cpp
#include <iostream>

void true_function() {}

class istream {
    class dummy {
    public:
        void true_function() {}
    };
public:
    bool fail() const {
        return true;
    }

    operator bool() {   // 1번
        return fail() ? false : true;
    }
    operator void*() {  // 2번
        return fail() ? nullptr : this;
    }
    typedef void(*PF)();
    operator PF() {     // 3번
        return fail() ? nullptr : &true_function;
    }
    typedef void(dummy::* PF2)();
    operator PF2() {    // 4번
        return fail() ? nullptr : &dummy::true_function;
    }
};
istream cin;

int main() {
    int n;
    // cin 을 scalar test 하고 싶다 if(cin) ..
    // 1. bool 로 변환하면 된다. 1번
    cin << n;   // cin 이 bool 로 변하면 << 연산자가 shift 연산자로 인식되어 에러가 나지 않는다.

    // 2. if() 에 놓이는데 << 는 에러가 나야한다: 포인터로 변환. 2번
    // 그런데, boost 팀에서 아래 문제점을 제시한다.
    delete cin;

    // 3. if() 에 놓이고 싶다. shift 연산 불가능. delete 안되어야 한다.
    // 함수포인터로 변환 하자. 3번
    void(*f)() = cin;
    
    // 4. if() 에 놓일수 있지만 최대한 side effect 를 줄이자. 4번
    void (istrema::dummy::*f)() = cin;
}
```