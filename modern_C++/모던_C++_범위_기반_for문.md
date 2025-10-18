# 모던 C++ 범위 기반 for문

## 범위 기반 for문

- 범위 기반 for 문을 쓰면 반복문이 더 간결해짐
- auto를 사용하면 자료형을 자동 추론
- 원본을 수정하려면 참조(&) 사용
- 읽기 전용이면 const auto& 사용

### 예시 코드
```cpp
#include <iostream>
#include <vector>

using namespace std;

int main() {
    vector<int> vec = {1, 2, 3, 4, 5};

    for (int num : vec) {
        cout << num << " ";   // 출력 값: 1 2 3 4 5
    }
    return 0;
}
```

### 예시 코드2
```cpp
#include <iostream>
#include <vector>

using namespace std;

int main() {
    vector<int> vec = {1, 2, 3, 4, 5};

    for (int& num : vec) {  // 참조 사용!
        num *= 2;  // 원본을 수정
    }

    for (const auto& num : vec) {  // 상수 참조 (값을 변경할 필요 없을 때)
        cout << num << " ";
    }

    return 0;
}
```

**출력 값:** 2 4 6 8 10

위에 있는건 범위 기반 for문을 사용한 것
근데 왜 똑같은 이름을 가진 변수가 다른 타입을 가지고 할 수 있느냐? 라고 할 수 있지만 저건 참조를 한것 

`int& num : vec`이 코드는 `&num = vec` 이거랑 똑같은 것 근데 for문은 다른 하나의 괄호로 닫혀있어서 for문이 나가면서 num이 소멸 됨 근데 vec은 참조를 하고 있기 때문에 값이 연결되서 값이 바뀔 수 있었음

```cpp
int num = 5;
{
    num += 2;
}

cout << num << endl;  // 출력 값: 5
```

약간 이런 느낌 괄호안에서 값을 더했지만 괄호 나가니까 소멸하는 것처럼 저것도 저런 것

## 범위 기반 for문은 인덱스관련되서 출력이 안됨

그니까 `vec[i]` 이런식으로가 안됨

## std::distance(begin, iter)

두 개의 반복자 사이의 거리를 계산하는 함수
`std::distance(시작 반복자, 현재 반복자);`
시작 반복자에서 현재 반복자 까지 거리를 구해
몇 번째 원소인지(인덱스) 알 수 있음

```cpp
vector<int> vec = {10, 20, 30, 40, 50};

auto it = vec.begin() + 3;  // 네 번째 원소(40)를 가리키는 반복자
int index = distance(vec.begin(), it);  // vec.begin()에서 it까지 거리

cout << "현재 원소: " << *it << endl;        // 40
cout << "현재 인덱스: " << index << endl; // 3
```

```cpp
for (auto it = vec.begin(); it != vec.end(); ++it) {
        int index = distance(vec.begin(), it);  // 현재 인덱스 계산
        cout << "vec[" << index << "] = " << *it << endl;
    }
```

**출력 값:**
```
10
20
30
40
50
```

## views::enumerate

범위 뷰를 만들어서 인덱스와 값을 출력 할 수 있기 만듬

```cpp
#include <iostream>
#include <vector>
#include <ranges>  // C++23부터 추가된 헤더

using namespace std;

int main() {
    vector<int> vec = {10, 20, 30, 40, 50};

    for (auto [i, num] : vec | views::enumerate) {
        cout << "vec[" << i << "] = " << num << endl;
    }
    return 0;
}
```

**출력 값:**
```
vec[0] = 10
vec[1] = 20
vec[2] = 30
vec[3] = 40
vec[4] = 50
```

위에 있는 i가 인덱스를 담당 그리고 num이 값을 담당
[인덱스, 값]

`vec | views::enumerate` 이거는 인덱스와 값을 모두 제공하는 범위

## 값 복사 vs 참조 vs 상수 참조

- **값 복사**: `for (auto num : vec)`는 vec의 각 원소를 복사합니다.
- **참조**: `for (auto& num : vec)`는 vec의 원소를 참조로 접근합니다.
- **상수 참조**: `for (const auto& num : vec)`는 vec의 원소를 상수 참조로 접근하여 값을 변경할 수 없습니다.

```cpp
#include <iostream>
#include <vector>

using namespace std;

int main() {
    vector<int> vec = {10, 20, 30, 40, 50};

    // 값 복사 (매우 비효율적, 큰 객체일수록 성능 저하)
    for (auto num : vec) {
        num += 10;  // vec의 원본 값은 변경되지 않음
    }

    // 참조 (값을 변경하지 않음)
    for (auto& num : vec) {
        num *= 2;  // vec의 원본 값을 변경
    }

    // 상수 참조 (값을 변경하지 않음)
    for (const auto& num : vec) {
        cout << num << " ";  // vec의 원본 값을 출력
    }

    return 0;
}
```
