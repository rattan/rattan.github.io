---
published: true
title: "rvalue"
date: 2022-05-10 21:43:02 +9:00
last_modified_at: 2022-05-10 21:43:03 +9:00
categories: cpp
---
lvalue
 - = 의 양쪽에 모두 올 수 있다.
 - 변수, 이름이 있다. 
 - 참조를 리턴하는 함수
 - 블럭을 벗어날 때 까지 생존
rvalue
 - = 의 오른쪽에만 올 수 있다.
 - 상수, 이름이 없다.
 - 값을 리턴하는 함수
 - 임시객체
 - 한 문장에서만 생존
```cpp
int x = 10;
int foo() {
	return x;
}
int& goo() {
	return x;
}

int main() {
	int n1 = 10;
	int n2 = 20;

	n1 = 10;
	10 = n1;	// error C2106: '=': 왼쪽 피연산자는 l-value이어야 합니다.
				// 10 은 rvalue 이다.
	n2 = n1;

	foo() = 20;	// error C2106: '=': 왼쪽 피연산자는 l-value이어야 합니다.
				// 값을 리턴 하는 함수. 임시객체 생성. rvalue
	goo() = 20;
}
```
[C2106 식이 수정할 수 있는 lvalue여야 합니다.]: https://docs.microsoft.com/en-us/cpp/error-messages/compiler-errors-1/compiler-error-c2106
> error code
>- [C2106 식이 수정할 수 있는 lvalue여야 합니다.][]
---
```cpp
struct Point {
	int x, y;
};
Point p;
Point foo() {
	return p;
}

int main() {
	int n1 = 10;
	int& r1 = n1;				// ok. c++ 참조는 lvalue 를 참조할 수 있다.
	int& r2 = 10;				// error C2440: '초기화 중': 'int'에서 'int &'(으)로 변환할 수 없습니다.
								// c++ 참조는 rvalue 를 참조 할 수 없다.

	Point p2;
	Point& r3 = p2;				// ok.
	Point& r4 = foo();			// error C2440: '초기화 중': 'Point'에서 'Point &'(으)로 변환할 수 없습니다.
								// 임시 객체는 rvalue 이다.

	// const& 는 rvalue 를 가르킬 수 있다.
	const int& r5 = 10;			// ok.
	const Point& r6 = foo();	// ok.
}
```
[C2440 A 형식의 값을 사용하여 B 형식의 엔터티를 초기화할 수 없습니다.]: https://docs.microsoft.com/ko-kr/cpp/error-messages/compiler-errors-1/compiler-error-c2440
> error code
>- [C2440 A 형식의 값을 사용하여 B 형식의 엔터티를 초기화할 수 없습니다.][]
---
std::sort( .. std::less()) 등의 표현식 처럼 임시객체를 인자로 보내려면 값 또는 const 참조 로 받아야 한다.<p>
const 참조의 함수 객체를 사용 하려면 ()연산자가 상수함수이어야 한다.<p>
함수 객체를 만들때 되도록이면 상수 함수로 하자.
```cpp
struct less {
	bool operator()(int a, int b) const {
		return a < b;
	}
};

template<typename T>
void sort(int* x, int n, const T& cmp) {
	cmp(x[0], x[1]);
}

int main() {
	int x[10] = { 0 };
	less f1;
	sort(x, 10, f1);

	sort(x, 10, less());	// 만들면서 인자로 전달: 임시 객체
}
```
---
 - &: lvalue 만 가르킨다.
 - const &: lvaue 와 rvalue 모두 가르킨다.
 - &&: rvalue 만 가르킨다. rvalue reference
```cpp
struct Point {
	int x, y;
};
Point p;
Point foo() {
	return p;
}

int main() {
	int n = 10;

	const int& r1 = n;
	const int& r2 = 10;

	int&& r3 = 10;				// ok. 임시 객체도 rvalue 이다.
	int&& r4 = n;				// error C2440: '초기화 중': 'int'에서 'int &&'(으)로 변환할 수 없습니다.
								// lvalue 를 담으려 하면 error.

	Point&& r5 = foo();			// ok. 임시 객체로 rvalue 이다.
								// 이 순간 임시 객체의 수명이 연장된다.
								// r5 를 선언한 블럭이 끝날 때 까지 살아 있다.

	const Point& r6 = foo();	// 이 순간에도 수명은 연장 되지만
								// 상수 객체인 상태로만 사용해야 한다.
}
```
---
함수 오버로딩과 참조
```cpp
#include <iostream>

void foo(int&) {		// 1번
	std::cout << "int&" << std::endl;
}
void foo(int&&) {		// 2번
	std::cout << "int&&" << std::endl;
}
void foo(const int&) {	// 3번
	std::cout << "const int&" << std::endl;
}

int main() {
	int n = 10;

	foo(n);				// int&
						// 1번, 없다면 3번
	foo(10);			// int&&
						// 2번, 없다면 3번

	int&& r = 10;
	foo(r);				// int&
						// rvalue reference 는 lvalue 이다.
						// rvalue 와 rvalue reference 를 혼동하기 쉽다.
}
```