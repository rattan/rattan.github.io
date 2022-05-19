---
title: "STL 컨테이너"
date: 2022-05-19 21:29:22 +9:00
last_modified_at: 2022-05-19 21:29:24 +9:00
categories: cpp
---
싱글 리스트를 생각해 보자
```cpp
#include <iostream>

template<typename T>
struct Node {
    T data;
    Node* next;
    Node(T a, Node* n) : data(a), next(n) {}
};

template<typename T>
class slist {
    Node<T>* head;
public:
    slist(): head(nullptr) {}

    // 아래처럼 Node 의 생성자를 잘 활용하면 싱글리스트 코드를 간단히 만들 수 있다.
    void push_front(const T& a) {
        head = new Node<T>(a, head);
    }
};

int main() {
    slist<int> s;
    s.push_front(10);
    s.push_front(20);
    s.push_front(30);
}
```
---
list 안에 있는 Node 를 가르키는 스마트 포인터를 도입 하자
 - ++로 이동하고 *로 값을 꺼낼 수 있게 하자

반복자
 - 컨테이너의 요소를 차례대로 이동하기 위한 객체
 - 스마트 포인터와 유사하지만 자원 해제가 아닌 요소의 열거
```cpp
#include <iostream>

template<typename T>
struct Node {
    T data;
    Node* next;
    Node(T a, Node* n) : data(a), next(n) {}
};

template<typename T>
class slist_iterator {
    Node<T>* current;
public:
    slist_iterator(Node<T>* p = nullptr) : current(p) {}

    // xfind 로 보내려면 진짜 포인터 처럼 동작해야 한다.
    // ++, *, ==, != 있어야 한다.
    inline slist_iterator& operator++() {
        current = current->next;
        return *this;
    }
    
    inline T& operator*() {
        return current->data;
    }

    inline bool operator==(const slist_iterator& t) {
        return current == t.current;
    }

    inline bool operator !=(const slist_iterator& t) {
        return current != t.current;
    }
};

template<typename T>
class slist {
    Node<T>* head;
public:
    slist(): head(nullptr) {}
    void push_front(const T& a) {
        head = new Node<T>(a, head);
    }

    // 모든 컨테이너 설계자는 자신의 반복자 이름을 미리 약속된 iterator 라는 이름으로 외부에 알려야 한다.
    typedef slist_iterator<T> iterator;

    // 모든 컨테이너는 자신의 시작 요소와 마지막 다음 요소를 가르키는 반복자를 리턴하는 함수를 제공해야 한다.
    iterator begin() {
        return iterator(head);
    }

    iterator end() {
        return iterator(nullptr);
    }
};

int main() {
    slist<int> s;
    s.push_front(10);
    s.push_front(20);
    s.push_front(30);

    slist<int>::iterator p = s.begin();
    // 이제부터는 p 를 s 의 처음 요소를 가르키는 포인터로 생각하면 된다.
    while (p != s.end()) {
        std:: cout << *p << std::endl;  // 30
                                                    // 20
                                                    // 10
        ++p;
    }

}
```