---
title: "raw string"
date: 2022-05-21 18:53:30 +9:00
last_modified_at: 2022-05-21 18:53:31 +9:00
categories: cpp
---
문자열 에서 \ 를 쓰고 싶을때: "\\\\"<p>
\ 를 한번만 적을 수 있는 표현식
```cpp
#include <iostream>

int main() {
    char s[] = "C:\\AAA\\BBB";
    std::cout<<s<<std::endl;    // C:\AAA\BBB

    char s2[] = R"(C:\AAA\BBB)";
    std::cout<<s2<<std::endl;   // C:\AAA\BBB
    
    // )" 가 끝나는 식별자 이므로 " 는 그냥 적을 수 있다.
    char s3[] =R"(C:\AA"A\BBB)";
    std::cout<<s3<<std::endl;   // C:\AA"\BBB
    
    // 문자열 중간에 )" 를 표시하고 싶을때는?
    char s4[] = R"(AAA)"BBB)";  // error.
    
    // 기본 식별자 "()" 에서 "와( 사이에 사용자가 추가 가능
    // "***()***"
    char s5[] = R"***(AAA)"BBB)***";
    std::cout<<s5<<std::endl;   // AAA)"BBB
}
```