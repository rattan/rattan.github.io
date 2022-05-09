---
published: true
title: "캐스팅"
date: 2022-05-09 21:36:47 +9:00
last_modified_at: 2022-05-09 21:36:48 +9:00
categories: cpp
---
c 캐스팅의 문제점
 - 대부분 성공한다
 - 너무 위험하고 버그가 많다
```cpp
#include <iostream>

int main() {
	int n = 10;
	double* p1 = &n;			// error C2440: '초기화 중': 'int *'에서 'double *'(으)로 변환할 수 없습니다.
								// 서로 다른 타입의 주소는 암시적 변환 할 수 없다.
								// c 에서는 가능하다.
	double* p2 = (double*)&n;	// 명시적 변환. ok
	*p2 = 3.4;

	const int c = 10;
	int* p3 = &c;				// error C2440: '초기화 중': 'const int *'에서 'int *'(으)로 변환할 수 없습니다.
								// 상수 주소는 비상수 포인터에 담을 수 없다.
	int* p4 = (int*)&c;			// 명시적 변환. ok
	*p4 = 20;

	std::cout << c << std::endl;// 10
}
```
[A 형식의 값을 사용하여 B 형식의 엔터티를 초기화할 수 없습니다.]: https://docs.microsoft.com/ko-kr/cpp/error-messages/compiler-errors-1/compiler-error-c2440
> error code
>- [A 형식의 값을 사용하여 B 형식의 엔터티를 초기화할 수 없습니다.][]
---
c++ 은 4개의 캐스팅 연산자를 지원한다
 - static_cast
   - void* => 다른 타입 은 허용
   - 그 외에는 연관성이 있어야 한다
 - reinterpret_cast
   - 메모리 재해석
   - 대부분 성공
 - const_cast
   - 객체의 상수 제거
 - dynamic_cast
```cpp
#include <iostream>

int main() {
	int* p1 = static_cast<int*>(malloc(100));	// void* => int* 변환

	int n = 10;
	double* p2 = (double*)&n;					// 명시적 변환. ok.
	double* p3 = static_cast<double*>(&n);		// error C2440: 'static_cast': 'int *'에서 'double *'(으)로 변환할 수 없습니다.
												// int* => double*
	double* p4 = reinterpret_cast<double*>(&n);	// ok

	const int c = 10;
	int* p5 = (int*)&c;							// 명시적 변환. ok.
	int* p6 = static_cast<int*>(&c);			// error C2440: 'static_cast': 'const int *'에서 'int *'(으)로 변환할 수 없습니다.
	int* p7 = reinterpret_cast<int*>(&c);		// error C2440: 'reinterpret_cast': 'const int *'에서 'int *'(으)로 변환할 수 없습니다.
												// 상수 제거는 안된다.
	int* p8 = const_cast<int*>(&c);				// ok. 상수성 제거

	*p8 = 20;									// 이렇게 사용되면 혼란스러워 질 수 있다.
}
```
---
```cpp
#include <iostream>

class X {
public:
	int x;
	char s;
	short k;
};

class Y {
public:
	int y;
};

class C : public X, public Y {};

int main() {
	C c;
	std::cout << &c << std::endl;		// 100 번지 라고 할 때
	Y* py1 = &c;						// 104 번지
	Y* py2 = (Y*)&c;					// 104 번지
	Y* py3 = static_cast<Y*>(&c);		// 104 번지
										// &c 메모리에서 Y 를 찾아라.
										// 없다면 error.
										// 컴파일 시간에 수행

	Y* py4 = reinterpret_cast<Y*>(&c);	// &c 주소를 무조건 Y* 로 생각하겠다
										// C 와 Y 의 연관성을 고려하지 않는다.
	py4->y = 10;
	std::cout << c.y << std::endl;		// 10
	std::cout << py4 << std::endl;		// 100 번지
}
```
---
dynamic_cast
 - 가상함수 테이블에 있는 type 정보를 사용 한다.
 - 가상함수가 반드시 1개 이상 있어야 한다.
c++ 에서 기본적으로 모든 부모의 소멸자는 가상함수 이어야 한다 는 규칙이 있으므로 대부분 한개 이상의 가상함수가 있다.
```cpp
#include <iostream>

class Animal {
public:
	virtual ~Animal() {}
};

class Dog : public Animal {
public:
	int color;
};

// 모든 동물에 공통의 기능만 수행 한다면, 부모인 Animal* 을 인자로.
void foo(Animal* p) {
	// 모든 동물의 공통 기능 수행 후

	// Dog 라면 색상도 변경
	// p->color = 10;	// 하지만 p 가 Animal* 이므로 color 가 없다.

	// 다운(down) 캐스트: 부모 포인터 => 자식 포인터 로 변경 하는 것
	// static_cast: 다운 캐스트 할 때 정말 Dog 인지를 확인 할 수 없다.
	//				컴파일 시간에 메모리를 조사할 수가 없다.
	 Dog* pDog1 = static_cast<Dog*>(p);
	 
	 // 실행 시간에 p 가 가르키는 메모리를 조사 해서 Dog 가 맞으면 성공. 아니면 0 리턴
	 Dog* pDog2 = dynamic_cast<Dog*>(p);
	 std::cout << pDog2 << std::endl;
	 if (pDog2 != 0) {
		 pDog2->color = 10;
	 }
}

void goo(Animal* p) {}	// 공통 작업만

// Dog 를 위한 작업을 추가 하려면 위의 goo 를 변경하는것 보다
// 아래처럼 만드는 것이 좋은 디자인 이다.
void goo(Dog* p) {
	goo(static_cast<Animal*>(p));   // 부모 타입으로 캐스팅 하여(업 캐스팅) goo 작업 후
	// Dog 만의 작업
}

int main() {
	Dog d;
	foo(&d);	// 0000007E0EEFFA18
	Animal a;
	foo(&a);	// 0
}
```