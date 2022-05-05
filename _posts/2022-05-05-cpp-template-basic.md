---
published: true
title: "템플릿 기본"
date: 2022-05-05 16:21:58 +9:00
last_modified_at: 2022-05-05 16:22:00 +9:00
categories: cpp
---
함수 오버로딩
 - 함수 사용자: 하나의 함수 처럼 보인다.
 - 함수 제작자: 2개의 함수를 만들어야 한다.
```cpp
int square(int a) {
    return a * a;
}
double square(double a) {
    return a * a;
}
```
유사한 코드가 반복 되면 코드 생성기를 사용하자
 1. 매크로에 의한 코드 생성: 전처리기가 코드 생성
    - 단점: 전처리기는 타입을 알지 못한다.
    - 어떤 타입의 함수가 필요한 지를 미리 선언해야 한다.
```cpp
#define MAKE_SQUARE(T) T square(T a) { return a * a; }
MAKE_SQUARE(int)
MAKE_SQUARE(double)
```
 2. 컴파일러에 의한 코드 생성
    - 컴파일러는 컴파일 중에 타입을 알 수 있다.
    - 어떤 타입의 함수가 필요한 지를 미리 알려 줄 필요가 없다.
```cpp
template<typename T>
T square(T a) {
    return a * a;
}
```
template instantiation(인스턴스 화)
 -  T 의 타입을 결정해서 실제 함수를 만드는 과정
암시적 인스턴스화
 - T 의 타입을 컴파일러가 결정(추론, Deduction) 해서 함수 생성
명시적 인스턴스화
 - T 의 타입을 사용자가 전달해서 함수 생성
 - 너무나 많은 함수/클래스가 생성되어 코드 메모리가 증가하는 현상
```cpp
template<typename T>
T square(T a) {
    return a * a;
}

int main() {
    square(3);      // int square(int a) { return a * a; } 함수 생성
    square(3.3);    // double square(aouble a) { return a * a; } 함수 생성
    
    short s = 3;
    square(s);      // short 버전의 함수 생성
    square<int>(s); // 사용자가 T 의 타입을 전달. int 형을 사용해 달라.
}
```
```cpp
??$square@N@@YANN@Z PROC				; square<double>, COMDAT
; 2    : T square(T a) {

	push	ebp
	mov	ebp, esp
	sub	esp, 8
??$square@H@@YAHH@Z PROC				; square<int>, COMDAT
; 2    : T square(T a) {

	push	ebp
	mov	ebp, esp
```
---
```cpp
#include <iostream>

template<typename T>
T square(T a) {
    return a * a;
}

int main() {
    std::cout<<&square<<std::endl;      // error C2568: '식별자': 함수 오버로드를 확인할 수 없습니다.
    std::cout<<&square<int><<std::endl; // ok.
}
```
[C2568 함수 템플릿 "square"의 인스턴스 중 사용할 인스턴스를 확인할 수 없습니다.]: https://docs.microsoft.com/ko-kr/cpp/error-messages/compiler-errors-2/compiler-error-c2568
> error code
>- [C2568 함수 템플릿 "square"의 인스턴스 중 사용할 인스턴스를 확인할 수 없습니다.][]
 - square: 함수를 만드는 틀
 - square<int>: 함수
---
지연된 인스턴스화: lazy instantiation
 - 사용되지 않은 함수(멤버함수) 템플릿은 인스턴스화(실제 함수로) 되지 않는다.
 - 이 개념을 사용한 기법이 아주 많이 있다.
```cpp
template<typename T>
class AAA {
public:
    void foo(T a) {
        *a = 10;    // T 의 타입이 포인터가 아니라면 error.
    }
};

int main() {
    AAA<int> aaa;   
    //aaa.foo(0);   // AAA<int>로 foo() 를 사용하지 않는다.
                    // void foo(int a) { *a = 10; } 이 생성되지 않는다.
}
```