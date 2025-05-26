---
published: true
title: "next permutation"
date: 2024-06-23 20:29:00
last_modified_at: 2024-06-23 20:29:00
categories: algorithm
---
대소 빅교가 가능한 원소를 가진 배열 에서 다음 순열을 구하는 알고리즘.

`[1,2,3]` 의 순열은 순서대로 아래와 같다.
```
[1,2,3]
[1,3,2]
[2,1,3]
[2,3,1]
[3,1,2]
[3,2,1]
```

배열 arr 가 주어졌을때,
 - arr[i] < arr[i + 1] 인 가장 큰 i 를 찾는다.
   - arr.size() - 2 위치 부터 0 까지, arr[i] < arr[i + 1] 인 위치가 있다면 i 선택.
   - i 를 찾을수 없다면 해당 배열은 내림차순으로 정렬되어 있다는 뜻으로, 마지막 번째의 순열이다. 이때는 배열을 뒤집어 주면 된다.
 - arr[i] < arr[j] 인 가장 큰 j 를 찾는다.
   - arr.size() - 1 위치 부터 i + 1 까지, arr[i] < arr[j] 인 위치가 있다면 j 선택.
 i 를 구할때 i 는 i + 1 보다 작은 값을 가지는 위치 이므로 j 는 항상 존재한다.
 - arr[i] 와 arr[j] 를 교환 한다.
 - i + 1 부터 배열의 끝까지 뒤집는다.

위 알고리즘을 `[4,7,5,9,6,3]` 에 적용해 보면,
```
     i
[4,7,5,9,6,3]
```
arr[i] < arr[i + 1] 인 i 를 찾는다.
```
     i   j
[4,7,5,9,6,3]
```
arr[i] < arr[j] 인 j 를 찾는다.
```
     i   j
[4,7,6,9,5,3]
```
두 위치의 값을 교환한다.
```
     i
[4,7,6,3,5,9]
```
i + 1 위치부터 끝까지 뒤집는다.

c++ code
``` c++
void next_permutation(std::vector<int> &arr) {
    int i = arr.size() - 2;
    for (; i >= 0; --i) {
        if (arr[i] < arr[i + 1]) {
            break;
        }
    }
    if (i != -1) {
        int j = arr.size() -1;
        for (;j > i; --j) {
            if (arr[i] < arr[j]) {
                break;
            }
        }
        int t = arr[i];
        arr[i] = arr[j];
        arr[j] = arr[i];
    }
    std::reverse(arr.begin() + i + 1, arr.end());
}
```
