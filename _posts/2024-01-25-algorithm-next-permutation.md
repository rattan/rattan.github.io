---
published: true
title: "Next permutation"
date: 2024-01-25 18:45:00
last_modified_at: 2024-01-25 18:45:00
categories: algorithm
---

### 다음 순열 구하기 (Next permutation)

주어진 수열의 다믕 번째 수열을 구하는 알고리즘

1. 수열의 오른쪽 끝에서 부터 arr[i] < arr[i + 1] 인 pivot i 위치를 구한다.
   - i 를 찾지 못하는 경우, 가장 큰 수열(내림차순으로 정렬) 으로, 전체 수열을 뒤집으면 다음 순열이 된다.
2. i 오른쪽 값들 중 arr[i] 보다 큰 값 중 가장 오른쪽 위치 j 를 찾는다.
3. j 를 찾는 경우, i 값과 j 값 을 교환한다.
4. i 이후 부터 오른쪽 끝까지의 수열을 뒤집는다.
---
구현 
1. 수열의 오른쪽 끝에서 부터 arr[i] < arr[i + 1] 인 pivot i 위치를 구한다.
```cpp
int i = arr.size() - 1, len = arr.size();
while (0 <= --i && arr[i + 1] < arr[i]);
```

i 를 찾지 못하는 경우, 가장 큰 수열(내림차순으로 정렬) 으로, 전체 수열을 뒤집으면 다음 순열이 된다.
```cpp
if (i == -1) {
    int sc = len >> 1;
    // reverse all arr
    while (sc) {
        int t = nums[i + sc];
        nums[i + sc] = nums[len - sc];
        nums[len - sc] = t;
        --sc;
    }
}
```

2. i 오른쪽 값들 중 arr[i] 보다 큰 값 중 가장 오른쪽 위치 j 를 찾는다.
```cpp
int j = len;
while (i < --j && nums[i] > nums[j]);
```

3. j 를 찾는 경우, i 값과 j 값 을 교환한다.
```cpp
int t = nums[i];
nums[i] = nums[j];
nums[j] = t;
```

4. i 이후 부터 오른쪽 끝까지의 수열을 뒤집는다.
```cpp
int sc = (len - i - 1) >> 1;
while (sc) {
    t = nums[i + sc];
    nums[i + sc] = nums[len - sc];
    nums[len - sc] = t;
    --rvs;
}
```
---
[std::next_permutation](https://github.com/gcc-mirror/gcc/blob/18a09944e8984be97a4aac004ee0a7ab10340526/libstdc%2B%2B-v3/include/bits/stl_algo.h#L2889)
```cpp
#include <algorithm>
std::next_permutation(first, last, comp)
```
stl algorithm 에도 동일한 방식의 알고리즘을 컨테이너에 사용할 수 있도록 구현 되어 있다.
