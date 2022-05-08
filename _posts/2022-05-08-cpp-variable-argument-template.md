---
published: true
title: "가변인자 템플릿"
date: 2022-05-08 19:26:52 +9:00
last_modified_at: 2022-05-08 19:26:54 +9:00
categories: cpp
---
가변인자 템플릿
```cpp
#include <iostream>

// 가변인자 템플릿. c++11 기술
template<typename ... Types>
void foo(Types ... args) {}

// 가변인자 클래스 템플릿
template<typename ... Types>
class Test {};

int main() {
    foo(1);
    foo(1, 3.3);    // Types: int, double
                    // args: 1, 3.3
    Test<int> t1;
    Test<int, double> t2;
}
```
---
```cpp
#include <iostream>

// 가변인자 템플릿. c++11 기술
template<typename ... Types>
void goo(Types ... args) {
    std::cout<<"goo"<<std::endl;
}

void hoo(int a, double d, char c) {
    std::cout<<"hoo"<<std::endl;
}

void joo(int *a, double *d, char *c) {
    std::cout<<"joo"<<std::endl;
}

// Types: 여러개의 타입을 가지고 있는 타입의 집합
// args: 여러개의 값을 가지고 있는 인자 집합: parameter pack
template<typename ... Types>
void foo(Types ... args) {
    // parameter pack 안에 인자 개수를 알고 싶다면
    std::cout<<sizeof...(args)<<std::endl;
    std::cout<<sizeof...(Types)<<std::endl;
    
    // parameter pack  을 다른 함수로 전달하는 방법
    goo(args...);
    hoo(args...);
    joo(&args...);
}

int main() {
    foo(1, 3.3, 'a');   // 3
                        // 3
                        // goo
                        // hoo
                        // joo
}
```
---
```cpp
#include <iostream>

int n = 0;

// 인자가 없는 경우
// 재귀를 멈출 오버로딩 함수가 먼저 정의되어 있어야 한다. 구현은 아래에 가능
// 이 함수를 아래. foo 밑에 정의하면 컴파일 error.
void foo() {}

// 인자 값을 꺼내려면 1번째 인자는 이름이 있는 변수로 받아야 한다.
// 결국 아래 template 은 foo() 함수를 4개 만들어 낸다.
template<typename T, typename... Types>
void foo(T value, Types... args) {
    ++n;
    std::cout<<n<<" "<<typeid(T).name()<<": "<<value<<std::endl;
    
    foo(args...);   // args: 3.3, 'a', "hello"
                    // args: 'a', "hello"
                    // args: "hello"
                    // args:
}

int main() {
    foo(1, 3.3, 'a', "hello");  // value: 1, args: 3.3, 'a', "hello"
                                // 1 i: 1
                                // 2 d: 3.3
                                // 3 c: a
                                // 4 PKc: hello
}
```
---
```cpp
#include <iostream>

template<typename T, typename... Types>
class xtuple: public xtuple<Types...> {
public:
    T value;
    
    xtuple() {}
    xtuple(const T &a, Types... args): value(a), xtuple<Types...>(args...) {}
    enum { N = xtuple<Types...>::N + 1};
};

// 이미 xtuple이 위에 있으므로, 새로운 xtuple 을 만들지 않고 부분 전문화 해야 한다.
// template<typename T> class xtuple: 새로운. xtuple
template<typename T>
class xtuple<T> {                       // 부분 전문화 버전
public:
    T value;
    xtuple() {}
    xtuple(const T &a): value(a) {}
    enum { N = 1 };
};

template<typename T>
int xtuple_size(T &a) {
    return T::N;
}


// xtuple 의 N 번째 요소의 타입을 알아내는 방법: 어렵다.
// 이 경우, primary 버전은 사용되지 않고 부분전문화 버전만 사용된다.
// 이 경우, 구조체 몸체{} 없이 선언만 있어도 된다.
// 단 선언 자체는 꼭 있어야 부분 전문화 버전을 만들 수 있다.
template<int N, typename T>
struct xtuple_type {
    typedef T type;
};

// 0 번째 타입을 요구 할 때를 위한 부분 전문화
template<typename T, typename... Types>
struct xtuple_type<0, xtuple<T, Types...>> {
    typedef T type;
    typedef xtuple<T, Types...> tuple_type;
};

// 0 이 아닌 경우
template<int N, typename T, typename... Types>
struct xtuple_type<N, xtuple<T, Types...>> {
    typedef typename xtuple_type<N-1, xtuple<Types...>>::type type;
    typedef typename xtuple_type<N-1, xtuple<Types...>>::tuple_type tuple_type;
};

// xtuple_type 을 확인하기 위한 함수
// VC 에서는 이 함수가 컴파일 잘 되는데 gcc 에서는 안된다...?
template<int N, typename T>
void print_type(const T &a) {
   std::cout<<typeid(xtuple_type<N, T>::type).name()<<std::endl;
}

template<size_t N, typename T>
xtuple_type<N, T>::type& xget(T& tp)
{
    return static_cast<typename xtuple_type<N, T>::tuple_type>(tp).value;
}
int main() {
    xtuple<int, char, double, short> t4(1, 'c', 3.3, 4);
    
    std::cout<<xtuple_size(t4)<<std::endl;  // 4
    
//    print_type<1>(t4);                      // char
    std::cout<<typeid(xtuple_type<0, xtuple<int, char, double, short>>::type).name()<<std::endl;
    // gcc 에서 이건 또 된다.
    // 함수 내부랑 컴파일 했을때 T 타입이 완전히 동일한데 왜 함수안에선 안되는지 모르겠다;
    
    std::cout<<xget<1>(t4)<<std::endl;
}
```
---