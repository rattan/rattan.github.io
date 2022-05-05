---
published: true
title: "람다"
date: 2022-05-04 22:19:58 +9:00
last_modified_at: 2022-05-04 22:20:00 +9:00
categories: cpp
---
인라인 함수와 포인터 관계
1. 인라인 치환은 컴파일 시간 문법 이다.
2. 인라인 함수라도 함수포인터에 담아서 사용하면 인라인 치환 되지 않는다.
```cpp
int add1(int a, int b) {
	return a + b;
}

inline int add2(int a, int b) {
	return a + b;
}

int main() {
	int n1 = add1(1, 2);	// 호출
	int n2 = add2(1, 2);	// 기계어 코드 치환
							// 속도가 빨라진다.
	int(*f)(int, int);
	f = &add2;
	int n3 = f(1, 2);		// 인라인 치환 안됨
}
```
VC++ 로 어셈블리 소스코드 만드는 법
 - cl lambda.cpp /Ob1 /FAs
   1. /Ob1: 인라인 치환 적용
   2. /FAs: 어셈블리 소스 생성
 - lambda.asm 에 컴파일된 어셈블리 코드 생성됨

g++ 로 어셈블리 소스코드 만드는 법
 - g++ lambda.cpp -S
 - lambda.s 에 컴파일된 어셈블리 코드 생성 됨
```cpp
; 10   : 	int n1 = add1(1, 2);	// 호출

	push	2
	push	1
	call	?add1@@YAHHH@Z				; add1
	add	esp, 8
	mov	DWORD PTR _n1$[ebp], eax

; 6    : 	return a + b;

	mov	eax, 1
	add	eax, 2

; 11   : 	int n2 = add2(1, 2);	// 기계어 코드 치환

	mov	DWORD PTR _n2$[ebp], eax

; 12   : 							// 속도가 빨라진다.
```
---
정책 변경이 가능한 함수 만들기

S/W 설계의 기본 원칙
 - 변하지 않는 전체 알고리즘 에서 변해야 하는 부분을 찾아서 분리 한다.

일반 함수는 변히는 부분을 함수 인자화 한다. (함수 포인터)

핵심은 "알고리즘과 정책의 분리"
```cpp
#include <algorithm>

void sort(int* x, int n, bool(*cmp)(int, int)) {	// c 표준 함수 qsort() 의 모양
	for (int i = 0; i < n - 1; ++i) {
		for (int j = i + 1; j < n; ++j) {
			if (cmp(x[i], x[j])) {
				std::swap(x[i], x[j]);
			}
		}
	}
}

inline bool cmp1(int a, int b) {					// sort() 사용자는 비교함수를 전달해야 한다.
	return a < b;
}
inline bool cmp2(int a, int b) {
	return a > b;
}

int main() {
	int x[10] = { 1,3,5,7,9,2,4,6,8,10 };

	sort(x, 10, cmp1);								// 동일한 함수를 다른 정책으로 사용 할 수 있게 된다.
	sort(x, 10, cmp2);
}
```
 - 장점: 함수가 사용하는 정책을 사용자가 변경 할 수 있다.
 - 단점: 결국 callback 함수를 사용하게 되므로 느리다.<p>
   정책함수를 인라인으로 만들어도 인라인 치환 되지 않는다.
---
() 연산자를 재정의 한 클래스: 함수 객체
```cpp
#include <iostream>

struct Plus {
	int operator()(int a, int b) {
		return a + b;
	}
};

int main() {
	Plus p;
	int n = p(1, 2);	// p.operator()(1, 2); 와 동일하다.
						// 결국 ()연산자만 재정의 하면 함수처럼 사용가능 하다.

	std::cout << n << std::endl;
}
```
---
왜 함수 객체를 사용하는가?
 1. 일반 함수는 자신만의 타입이 없다.
    - signiture 가 동일한 함수는 같은 타입이다.
 2. 함수 객체는 자신만의 타입이 있다.
    - signiture 가 동일해도 모든 함수 객체는 다른 타입이다.
```cpp
#include <algorithm>

struct less {
	inline bool operator()(int a, int b) {
		return a < b;
	}
};
struct greater {
	inline bool operator()(int a, int b) {
		return a > b;
	}
};

void sort(int* x, int n, less cmp);	// 이 함수는 cmp() 가 인라인 치환 되지만 greater 로 교체 할 수 없다.

template<typename T>
void sort(int* x, int n, T cmp) {	// 정책 교체가 가능하고 정책이 인라인 치환 되는 함수
									// 템플릿 + 함수 객체 를 이용한 기술
	for (int i = 0; i < n - 1; ++i) {
		for (int j = i + 1; j < n; ++j) {
			if (cmp(x[i], x[j])) {	// 인라인 치환 가능.
				std::swap(x[i], x[j]);
			}
		}
	}
}

inline bool cmp1(int a, int b) {
	return a < b;
}
inline bool cmp2(int a, int b) {
	return a > b;
}

#include <iostream>
#include <algorithm>				// 정책 변경이 가능한 std::sort() 가 여기 있다.
									// std::sort() 의 모든 인자는 템플릿이다.

int main() {
	int x[10] = { 1,3,5,7,9,2,4,6,8,10 };

	std::sort(x, x + 10, cmp1);		// 비교 정책으로 일반 함수를 사용하는 경우
	std::sort(x, x + 10, cmp2);		// 장점: 정책을 여러번 교체 해도 std::sort() 기계어는 하나만 만들어 진다.
									// 메모리 사용량 감소
									// 단점: 정책이 인라인 치환 될 수 없기 때문에 성능 저하가 있다.
									// std::sort(int*, int *, bool(*)(int, int)) 함수 생성

	less f1;
	greater f2;
	std::sort(x, x + 10, f1);		// 비교정책으로 함수 객체를 사용하는 경우
	std::sort(x, x + 10, f2);		// 장점: 정책이 인라인 치환 되므로 속도가 빠르다.
									// 단점: 정책을 교체한 횟수 만큼의 std::sort() 함수 생성
									// 코드 메모리가 증가 한다.

	std::sort(x, x + 10, [](int a, int b) { return a < b; });
									// c++11/14 람다 표현식 (lambda expression)
									// 함수 인자로 함수의 구현(코드) 를 전달하는 기술
									// []: lambda introducer

}
```
---
람다 표현식의 정체
```cpp
#include <algorithm>

int main() {
	int x[10] = { 1,3,5,7,9,2,4,6,8,10 };
	
	std::sort(x, x + 10, [](int a, int b) { return a < b; });

	class Closure_Object {					// 위 람다식을 보고 컴파일러는 함수 객체를 생성 한다.
	public:
		inline bool operator()(int a, int b) const {
			return a < b;
		}
	};

	std::sort(x, x + 10, Closure_Object());	// 임시객체로 전달
}
```
---
람다와 타입
```cpp
#include <iostream>

int main() {
	auto f1 = [](int a, int b) { return a + b; };	// 람다는 auto 에 담을 수 있다.
	auto f2 = [](int a, int b) { return a + b; };	// f1, f2 는 같은 타입일까?

	std::cout << typeid(f1).name() << std::endl;	// class <lambda_64ed94019a426eb47772208022ae9800>
	std::cout << typeid(f2).name() << std::endl;	// class <lambda_1b29d4288c38e42585593292d7a463ec>
													// RTTI 기술로 컴파일러가 만든
													// 함수 객체 이름을 확인 할 수 있다.
	std::cout << f1(1, 2) << std::endl;				// 3
}
```
---
```cpp
#include <iostream>
#include <functional>

int main() {
	// 람다는 3가지 형태의 변수에 담을 수 있다.
	auto f1 = [](int a, int b) { return a + b; };
	int(*f2)(int, int) = [](int a, int b) { return a + b; };
	std::function<int(int, int)> f3 = [](int a, int b) { return a + b; };

	f1(1, 2);								// 인라인 치환 가능.
	f2(1, 2);								// 인라인 치환 불가능.
	f3(1, 2);								// 인라인 치환 불가능.

	foo([](int a, int b) {return a + b; });
}

// 람다를 인자로 받는 방법
void foo(int(*f2)(int, int));				// ok.
void foo(std::function<int(int, int)> f3);	// ok.

// auto 는 함수 인자가 될 수 없다.
void foo(auto f1);							// error C3533: 매개 변수에 'auto'이(가) 포함된 형식을 포함할 수 없습니다.

template<typename T>						// 인라인 치환 가능하게 람다를 받는 유일한 방법: template
void foo(T f) {
	f(1, 2);
}
```
> error code
>- [C3533 여기에는 'auto'를 사용할 수 없습니다.](https://docs.microsoft.com/ko-kr/cpp/error-messages/compiler-errors-2/compiler-error-c3533){:target="_blank"}
---
람다와 리턴 타입
```cpp
int main() {
	// 리턴 타입의 인자() 뒤에 적는 문법: traling return
	auto f1 = [](int a, int b) -> int { return a + b; };

	auto f2 = [](int a, int b) { return a > b ? a : b; };

	auto f3 = [](int a, int b) -> double { return a > b ? a : 3.0; };
}
```
---
캡쳐
```cpp
#include <iostream>

int main() {
	int v1 = 10;
	int v2 = 20;

	// 지역변수 캡쳐
	auto f1 = [v1](int a, int b) { return a + b + v1; };
	auto f2 = [v1, v2](int a, int b) { return a + b + v2; };
	auto f3 = [=](int a, int b) { return a + b + v1 + v2; };

	std::cout << f1(1, 2) << std::endl;	// 13
	std::cout << f2(1, 2) << std::endl;	// 23
	std::cout << f3(1, 2) << std::endl;	// 33

	// 참조에 의한 캡쳐
	auto f4 = [&v1](int a, int b) { v1 = 0; return a + b; };
	auto f5 = [&](int a, int b) { v2 = 0; return a + b; };

	f4(1, 2);
	f5(1, 2);
	std::cout << v1 << std::endl;		// 0
	std::cout << v2 << std::endl;		// 0


}
```
---
캡쳐의 원리
```cpp
#include <iostream>

int main() {
	int v1 = 10;
	int v2 = 20;

	auto f1 = [v1](int a, int b) { return a + b + v1; };

	class Closure_object {
		int value1;
	public:
		Closure_object(int v1) : value1(v1) {}
		inline int operator()(int a, int b) const{
			return a + b;
		}
	};

	auto f = Closure_object(v1);	// 지역변수를 값으로 전달 한다.
}
```
---
멤버변수 캡쳐
```cpp
class Test {
	int base;
public:
	Test(int n) : base(n) {}
	void foo() {
		// 멤버 변수인 base 를 캡쳐 하려면?
		auto f1 = [base]() { std::cout << base << std::endl; };	// error C3480: 'Test::base': 람다 캡처는 바깥쪽 함수 범위에 속한 변수여야 합니다.
		auto f2 = [this]() { base = 200; };

		class Closure_class {
			Test* value;
		public:
			Closure_class(Test* v) :value(v) {}					// 실제 this 포인터가 넘어간다.
			inline void operator()() const {
				value->base = 200;								// this 의 멤버 변경 가능
			}
		};

		f2();
	}
};

int main() {
	Test t(20);
	t.foo();
}
```
> error code
>- [C3480 멤버 "Test::base"은(는) 변수가 아닙니다.](https://docs.microsoft.com/ko-kr/cpp/error-messages/compiler-errors-2/compiler-error-c3480){:target="_blank"}
---
인자 없는 람다
```cpp
#include <iostream>

int main() {
	auto f1 = []() {
		std::cout << "f1" << std::endl;
	};
	auto f2 = [] {	// 인자가 없는 람다는 () 생략이 가능하다
					// nullary lambda expression
		std::cout << "f2" << std::endl;
	};

	f1();			// f1
	f2();			// f2
}
```
---
```cpp
#include <iostream>

int main() {
	int v = 10;
	auto f1 = [&v] { v = 0; };			// 참조로 캡쳐
	class Closure_class1 {
		int &value;						// 캡쳐할 값을 참조
	public:
		Closure_class1(int& v) : value(v) {}
		inline void operator()() const {
			value = 0;
		}
	};

	auto f2 = [v] { v = 0; };			// error C3491: 'v': 변경 불가능한 람다에서 복사 방식 캡처를 수정할 수 없습니다.
	class Closure_class2 {
		int value;						// 캡쳐할 값을 저장
	public:
		Closure_class2(int& v) : value(v) {}
		inline void operator()() const {
			value = 0;					//  error C3490: 'value'은(는) const 개체를 통해 액세스되고 있으므로 수정할 수 없습니다.
										// const 함수에서 멤버를 변경할 수 없다.
		}
	};

	auto f3 = [v]() mutable { v = 0; };	// mutable lambda: 캡쳐한 복사본의 변경이 가능한다.
	class Closure_class3 {
		int value;						// 캡쳐할 값을 저장
	public:
		Closure_class3(int& v) : value(v) {}
		inline void operator()() {		// const 함수가 아닌 일반 함수가 만들어 진다.
			value = 0;
		}
	};
}
```
> error code
>- [C3491 식이 수정할 수 있는 lvalue여야 합니다.](https://docs.microsoft.com/ko-kr/cpp/error-messages/compiler-errors-2/compiler-error-c3491){:target="_blank"}
>- [C3490 식이 수정할 수 있는 lvalue여야 합니다.](https://docs.microsoft.com/ko-kr/cpp/error-messages/compiler-errors-2/compiler-error-c3490){:target="_blank"}