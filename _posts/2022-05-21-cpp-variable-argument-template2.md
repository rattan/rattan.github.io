---
title: "가변인자 템플릿2"
date: 2022-05-21 20:16:50 +9:00
last_modified_at: 2022-05-21 20:16:51 +9:00
categories: cpp
---
```cpp
#include <iostream>

int negative(int n) {
    return -n;
}

void goo(int a, int b, int c) {
    std::cout<<a<<", "<<b<<", "<<c<<std::endl;
}

template<typename ...Types>
void foo(Types... args) {
    goo(args...);           // goo(1, 2, 3);
    goo(negative(args...)); // error.
                            // negative 는 인자가 한개 이다.
    goo(negative(args)...); // goo(negative(1), negative(2), negative(3));
}

int main() {
    foo(1, 2, 3);
}
```