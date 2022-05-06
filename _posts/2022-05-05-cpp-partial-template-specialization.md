---
published: true
title: "템플릿 부분 전문화"
date: 2022-05-05 20:12:57 +9:00
last_modified_at: 2022-05-06 15:46:09 +9:00
categories: cpp
---
템플릿 부분 전문화
```cpp
#include <iostream>

template<typename T>
class stack {
    T buff[100];
public:
    void push() {
        std::cout<<"T"<<std::endl;
    }
};

// partil specialization, 부분 전문화, 부분 특화
template<typename T>
class stack<T*> {
    T buff[100];
public:
    void push() {
        std::cout<<"T*"<<std::endl;
    }
};

// specialization, 전문화
template<>
class stack<char*> {
    char* buff[100];
public:
    void push() {
        std::cout<<"char*"<<std::endl;
    }
};

int main() {
    stack<int> s1;
    s1.push();      // T
    stack<int*> s2;
    s2.push();      // T*
    stack<char*> s3;
    s3.push();      // char*
}
```
---
template meta programming: 컴파일 시에 연산을 수행 하는것
 - 컴파일 시간 재귀를 활용 한다.
 - 재귀의 종료를 위해서 전문화 문법이 활용된다.
 - 메타 함수 라고 한다.
```cpp
#include <iostream>

template<int N>
struct Factorial {
    enum { value = N * Factorial<N-1>::value };
};

template<>
struct Factorial<1> {
    enum { value = 1 };
};

template<int N>
struct Binary {
    // 입력되는 값중 0, 1 이 아닌 값이 있으면, 컴파일 에러를 발생 시킨다.
    typedef char bianry_digit_not_allow_2_to_9[N % 10 > 1 ? -1: 0];
    enum { value = (N & 1) | Binary<(N/10)>::value<<1 };
};

template<>
struct Binary<1> {
    enum { value = 1 };
};

template<>
struct Binary<0> {
    enum { value = 0 };
};

int main() {
    int n = Factorial<5>::value;    // 5 * F<4>::v
                                    // 5 * 4 * F<3>::v
                                    // 5 * 4 * 3 * F<2>::v
                                    // 5 * 4 * 3 * 2 * F<1>::v
                                    // 5 * 4 * 3 * 2 * 1
    std::cout<<n<<std::endl;        // 120
    
    int a = 5;
    std::cin>>a;
    int n2 = Factorial<a>::value;   // error
                                    // 템플릿 인자는 컴파일시간 상수만 가능
    
    int n3 = Binary<101>::value;
    std::cout<<n3<<std::endl;       // 5
}
```
---
```cpp
#include <iostream>

template<typename T>
void print(const T &d) {
    std::cout<<T::N<<std::endl;
}

template<typename T, typename U>
struct Duo {
    T value1;
    U value2;
    enum { N = 2 };
};

//  Duo 의 2번쨰 인자가 다시 Duo 일때(Recursive) 를 위한 부분 전문화
// 부분 전문화 할때 템플릿 인자 개수가 바뀌어도 된다.
template<typename A, typename B, typename C>
struct Duo<A, Duo<B, C>> {
    A value1;
    Duo<B, C> value2;
    enum { N = Duo<B, C>::N + 1 };
};

// 1번째가 다시 Duo 일 때
template<typename A, typename B, typename C>
struct Duo<Duo<A, B>, C> {
    Duo<A, B> value1;
    C value2;
    enum { N = Duo<A, B>::N + 1 };
};
// 1, 2 모든 인자가  Duo일 때
template<typename A, typename B, typename C, typename D>
struct Duo<Duo<A, B>, Duo<C, D>> {
    Duo<A, B> value1;
    Duo<A, B> value2;
    enum { N = Duo<A, B>::N + Duo<C, D>::N };
};

int main() {
    // 1번째 인자가 Duo 일 때
    Duo<Duo<int, int>, int> d1;
    Duo<Duo<Duo<int, int>, int>, int> d2;
    print(d1);  // 3
    print(d2);  // 4
    
    // 2번째 인자가 Duo 일 때
    Duo<int, Duo<int, int>> d3;
    Duo<int, Duo<int, Duo<int, int>>> d4;
    print(d3);  // 3
    print(d4);  // 4
    
    // 1, 2번쨰 인자가 Duo 일 때
    Duo<Duo<int, int>, Duo<int, int>> d5;
    print(d5);  // 4
}
```
---
```cpp
template<typename T, typename U>
struct Duo {
    T value1;
    U value2;
    enum { N = 2 };
};

// Recursive Duo 를 사용하기 쉽게 선형화 하는 코드
struct NullType {};

// xtuple의 1번째 타입을 Duo 의 1번째 타입으로
// xtuple의 2~ 번째 타입을 xtuple 로 만들어 Duo 의 2번째 타입으로 만든다.
template<typename P1 = NullType,
        typename P2 = NullType,
        typename P3 = NullType,
        typename P4 = NullType,
        typename P5 = NullType>
class xtuple: public Duo<P1, xtuple<P2, P3, P4, P5, NullType>> {
};

// xtuple 이 2개의 유효한 타입일때를 위한 부분 전문화
template<typename A, typename B>
class xtuple<A, B, NullType, NullType, NullType>: public Duo<A, B> {
};

int main() {
    xtuple<int, char, short, long, double> t5;
    // Duo<int, xtuple<char, short, long, double, N>
    //          Duo<char, xtuple<short, long, double, N, N>
    //                    Duo<short, xtuple<long, double, N, N, N>
    //                               Duo<long, double>
    
    xtuple<int, int, int> t3;
    // Duo<int, xtuple<int, int, N, N, N>
    //          Dou<int, int>
}
```
---
tuple 은 이미 c++표준에 있다.
 - ~c++99: 10개 까지 가능, 우리가 만든 코드와 유사한 기술
 - c++11: c++11 의 가변인자 기술로 이론상 무한히 가능
```cpp
#include <iostream>
#include <tuple>

int main() {
    int x[10];                          //같은 타입 10개
    
    std::tuple<int, double, long> t3(1, 3.3, 2);
    
    std::cout<<get<1>(t3)<<std::endl;   // 3.3
                                        // tuple 에서 값 꺼내는 법
}
```
---
```cpp
#include <iostream>

// int 2개를 받아 곱하여 int 를 리턴
int Mul(int a, int b) {
    return a * b;
}

// T 2개를 받아 곱하여 T 를 리턴
template<typename T>
T Mul(T a, T b) {
    return a * b;
}

// 두개의 타입을 받았을때 어떤 타입으로 리턴해야 할까?
//template<typename T, typename U>
//? Mul(T a, U b) {
//    return a * b;
//}

// bool 기반 type selection 기술
// primary template 의 인자가 3개이다.
// 1번째 타입을 결과 타입으로 정한다.
template<bool, typename T, typename U>
struct IfThenElse {
    typedef T resultT;
};

// IfThenElse 의 bool 이 false 이면, 2번째 타입을 결과타입으로 정한다.
template<typename T, typename U>
struct IfThenElse<false, T, U> {
    typedef U ResultT;
};

// 1, 2번째 인자 중 자료형의 크기가 큰 값으로 결과타입을 정한다.
template<typename T, typename U>
typename IfThenElse<(sizeof(T) > sizeof(U)), T, U>::resultT
Mul(T a, U b) {
    return a * b;
}
// c++11 기술
// 람다와 유사한 trailing return 기술
template<typename T, typename U>
auto Mul2(T a, U b) -> decltype(a * b) {
    return a * b;
}

int main() {
    std::cout<<Mul(3.3, 2)<<std::endl;  // 6.6
    std::cout<<Mul2(2, 3.3)<<std::endl; // 6.6
}
```