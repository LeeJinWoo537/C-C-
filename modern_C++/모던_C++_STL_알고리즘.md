# 모던 C++ STL 알고리즘

## STL(컨테이너) 알고리즘

- 데이터를 저장하고 관리하는 구조체들이야. C++에서는 여러 가지 컨테이너를 제공해서 다양한 방식으로 데이터를 저장하고 조작할 수 있어.

### 3가지로 나뉠 수 있는데

#### 시퀸스(순차) 컨테이너
**특징:** 데이터를 순차적으로 저장함

- **vector**: 동적 배열, 크기가 자동으로 증가.
- **deque**: 양쪽 끝에서 빠르게 삽입/삭제 가능.
- **list**: 이중 연결 리스트, 중간 삽입/삭제 빠름.
- **forward_list**: 단일 연결 리스트, 메모리 절약.

**언제 사용?**
- 요소 개수가 동적으로 변할 때 → vector
- 중간 삽입/삭제가 자주 발생할 때 → list
- 앞뒤 삽입/삭제가 많을 때 → deque

#### 연관 컨테이너 (Associative Containers)
**특징:** 키(key) 기반으로 데이터 저장. 검색/정렬 속도가 빠름.

**종류:**
- **set**: 중복 허용 안됨, 자동 정렬됨.
- **multiset**: 중복 허용, 자동 정렬됨.
- **map**: key-value 형태, 중복된 키 없음.
- **multimap**: key-value 형태, 중복된 키 허용.

**언제 사용?**
- 중복 없는 데이터 저장 → set
- key-value로 빠르게 찾기 → map

#### 비정렬 연관 컨테이너 (Unordered Containers)
**특징:** hash table을 사용해서 빠른 검색이 가능.

**종류:**
- **unordered_set**: set과 같지만 정렬되지 않음.
- **unordered_map**: map과 같지만 정렬되지 않음.
- **unordered_multiset**: multiset과 같음.
- **unordered_multimap**: multimap과 같음.

**언제 사용?**
- 정렬 필요 없이 빠른 검색이 필요할 때 → unordered_map

### 시퀸스 컨테이너 종류
- vector
- deque
- list
- forward_list
- array

## vector (동적 배열)

**특징:** 일반 배열과 유사하지만 크기가 자동 조정됨

**장점:**
- 랜덤 접근 (O(1))
- 메모리 연속적 → 캐시 효율 높음
- push_back()으로 동적 추가 가능

**단점:**
- 중간 삽입/삭제 시 느림 (O(N))
- 크기 변경 시 메모리 재할당 발생 가능

### 예시 코드
```cpp
#include <iostream>
#include <vector>
using namespace std;

int main() {
    vector<int> v = {1, 2, 3};
    v.push_back(4);  // 값 추가
    v.push_back(5);

    cout << "Vector: ";
    for (int num : v) cout << num << " ";
    cout << endl;

    cout << "세 번째 원소: " << v[2] << endl;  // 랜덤 접근 가능
}
```

**언제 사용?**
배열처럼 사용하면서 크기를 유동적으로 조정하고 싶을 때!

**특징**
- 동적 크기 조절 가능 → 원소를 추가하면 자동으로 크기가 증가
- 배열처럼 인덱스로 접근 가능 → O(1) 연산
- 연속적인 메모리 블록 사용 → 캐시 효율이 높음
- 빠른 삽입 및 삭제 (끝 부분) → push_back(), pop_back()
- 중간 삽입/삭제는 비효율적 → insert(), erase()는 O(N)

### vector 코드 사용법
```cpp
#include <iostream>
#include <vector>

int main() {
    std::vector<int> v;  // 정수형 벡터 선언

    // 요소 추가
    v.push_back(10);
    v.push_back(20);
    v.push_back(30);

    // 요소 접근
    std::cout << "첫 번째 요소: " << v[0] << std::endl;
    std::cout << "두 번째 요소: " << v.at(1) << std::endl;

    // 크기 확인
    std::cout << "벡터 크기: " << v.size() << std::endl;    // 백터 크기: 3

    // 마지막 요소 삭제
    v.pop_back();
    std::cout << "마지막 요소 삭제 후 크기: " << v.size() << std::endl;   // 2

    return 0;
}
```

### 주요 함수들
| 함수 | 설명 |
|------|------|
| `push_back(value)` | 맨 뒤에 값 추가 |
| `pop_back()` | 맨 뒤의 값 삭제 |
| `size()` | 현재 벡터의 크기 반환 |
| `capacity()` | 할당된 메모리 크기 반환 |
| `at(index)` | 해당 인덱스의 값 반환 (범위 체크 O) |
| `operator[]` | 해당 인덱스의 값 반환 (범위 체크 X) |
| `insert(it, value)` | 특정 위치에 값 삽입 |
| `erase(it)` | 특정 위치의 요소 삭제 |
| `clear()` | 모든 요소 삭제 |
| `empty()` | 벡터가 비어있는지 확인 |
| `resize(n)` | 크기 변경 |
| `reserve(n)` | 메모리 미리 할당 |

### 반복문 사용
```cpp
#include <iostream>
#include <vector>

int main() {
    std::vector<int> v = {1, 2, 3, 4, 5};

    // 1️⃣ 일반 for문
    for (size_t i = 0; i < v.size(); i++) {
        std::cout << v[i] << " ";
    }
    std::cout << std::endl;

    // 2️⃣ 범위 기반 for문 (C++11 이상)
    for (int num : v) {
        std::cout << num << " ";
    }
    std::cout << std::endl;

    // 3️⃣ 반복자 사용
    for (auto it = v.begin(); it != v.end(); it++) {
        std::cout << *it << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

### std::vector와 메모리 관리 (reserve vs resize)
```cpp
#include <iostream>
#include <vector>

int main() {
    std::vector<int> v;
    
    // 메모리 미리 할당 (capacity 증가)
    v.reserve(10);

    // 크기 변경 (실제 요소도 생성됨)
    v.resize(5);

    std::cout << "size: " << v.size() << ", capacity: " << v.capacity() << std::endl;

    return 0;
}
```

- `reserve(n)` → 메모리만 확보, 실제 크기는 변하지 않음
- `resize(n)` → 메모리를 늘리고 새로운 요소를 0으로 초기화

### resize(n)의 동작 방식
- 현재 크기보다 작게 줄이면 뒤의 요소를 삭제
- 현재 크기보다 크게 늘리면 새로운 요소를 0으로 초기화
- 메모리를 다시 할당할 수도 있음 (기존 capacity보다 커질 경우)

### 코드 예제
```cpp
#include <iostream>
#include <vector>

int main() {
    std::vector<int> v = {1, 2, 3};

    v.resize(5);  // 크기 증가 → 새로운 요소는 0으로 초기화
    for (int num : v) {
        std::cout << num << " ";  // 출력: 1 2 3 0 0
    }
    std::cout << std::endl;

    v.resize(2);  // 크기 감소 → 3, 0, 0 삭제
    for (int num : v) {
        std::cout << num << " ";  // 출력: 1 2
    }
    std::cout << std::endl;

    return 0;
}
```

### reserve(n): 메모리 미리 할당
- `reserve(n)`은 벡터의 capacity(할당된 메모리 크기)를 증가시키지만 size는 그대로 유지해.
- 즉, 크기는 변하지 않지만, 더 많은 요소를 추가할 수 있도록 메모리를 미리 확보하는 기능이야.

### reserve(n)의 필요성
벡터의 크기가 늘어날 때마다 새로운 배열을 할당 → 기존 데이터를 복사하는 과정이 발생하는데, reserve()를 사용하면 메모리 재할당을 줄여 성능을 최적화할 수 있어.

```cpp
#include <iostream>
#include <vector>

int main() {
    std::vector<int> v;
    v.reserve(3);  // 메모리만 3개 크기로 확보

    std::cout << "size: " << v.size() << ", capacity: " << v.capacity() << std::endl;  // 0   3
    
    v.push_back(1);
    v.push_back(2);
    /*v.push_back(3);
    //v.push_back(4);    //push_vack 3, 4를 추가하면 메모리가 2배로 증가한다
    std::cout << "size: " << v.size() << ", capacity: " << v.capacity() << std::endl;   // 4   6 */
    std::cout << "size: " << v.size() << ", capacity: " << v.capacity() << std::endl;   // 2   3

    return 0;
}
```

### resize() vs reserve() 차이점 정리
| 함수 | 크기(size) 변화 | 메모리(capacity) 변화 | 요소 초기화 여부 |
|------|----------------|---------------------|-----------------|
| `resize(n)` | O | O | 새 요소는 0으로 초기화 |
| `reserve(n)` | X | O | 초기화 안 됨 |

| 함수 | 기능 | 언제 사용? |
|------|------|-----------|
| `resize(n)` | 크기를 변경하고, 필요하면 새 요소를 0으로 초기화 | 특정 크기로 맞춰야 할 때 |
| `reserve(n)` | 메모리만 미리 할당 (크기 변화 X) | 많은 데이터를 추가할 때 성능 최적화 |

- 배열처럼 크기를 정하고 사용하려면 resize(n) 사용
- 많은 데이터를 넣을 예정이면 reserve(n)로 메모리 미리 확보

### 최적화 예제 (reserve()를 사용하면 성능 향상!)
```cpp
#include <iostream>
#include <vector>

int main() {
    std::vector<int> v;
    v.reserve(1000);  // 미리 메모리 할당

    for (int i = 0; i < 1000; i++) {
        v.push_back(i);
    }

    std::cout << "size: " << v.size() << ", capacity: " << v.capacity() << std::endl;
}
```

이렇게 하면 push_back()할 때 메모리 재할당이 줄어들어 성능이 좋아져!

## deque (양쪽 삽입/삭제 빠름)

**특징:** 양쪽 끝에서 삽입/삭제가 빠름.

**장점:**
- 앞뒤 삽입/삭제 O(1)
- vector보다 중간 삽입이 빠름

**단점:**
- 메모리 구조가 조각날 수 있음 (연속적이지 않음)

### 예시 코드
```cpp
#include <iostream>
#include <deque>
using namespace std;

int main() {
    deque<int> d = {2, 3, 4};
    d.push_front(1);  // 앞에 삽입
    d.push_back(5);   // 뒤에 삽입

    cout << "Deque: ";
    for (int num : d) cout << num << " ";
    cout << endl;
}
```

**언제 사용?**
- 앞뒤에서 빠르게 삽입/삭제해야 할 때!

양쪽(앞/뒤)에서 삽입과 삭제가 빠른 컨테이너야.
std::vector와 std::list의 장점을 조합한 컨테이너라고 볼 수 있어!

### std::deque의 특징
- 양방향 삽입/삭제 O(1): 앞과 뒤에서 push/pop이 빠름
- 임의 접근 O(1): operator[] 사용 가능 (벡터처럼 사용 가능)
- 중간 삽입/삭제 O(n): 중간 요소 추가/삭제는 성능이 떨어짐
- 동적 배열이지만 블록 단위로 관리: 벡터는 한 번에 큰 메모리를 할당하지만, 덱은 작은 블록을 연결해서 사용

### deque 코드 사용법
```cpp
#include <iostream>
#include <deque>

int main() {
    std::deque<int> dq = {10, 20, 30};

    dq.push_back(40);  // 뒤에 추가
    dq.push_front(5);  // 앞에 추가

    std::cout << "첫 번째 요소: " << dq.front() << std::endl;  // 5
    std::cout << "마지막 요소: " << dq.back() << std::endl;   // 40

    dq.pop_front();  // 앞에서 삭제
    dq.pop_back();   // 뒤에서 삭제

    for (int num : dq) {
        std::cout << num << " ";  // 출력: 10 20 30
    }

    return 0;
}
```

### 주요 함수들
| 함수 | 설명 |
|------|------|
| `push_front(val)` | 앞에 요소 추가 |
| `push_back(val)` | 뒤에 요소 추가 |
| `pop_front()` | 앞 요소 제거 |
| `pop_back()` | 뒤 요소 제거 |
| `front()` | 첫 번째 요소 반환 |
| `back()` | 마지막 요소 반환 |
| `operator[]` | 인덱스로 접근 |
| `size()` | 요소 개수 반환 |
| `clear()` | 모든 요소 제거 |
| `insert(it, val)` | 특정 위치에 값 삽입 |
| `erase(it)` | 특정 위치의 요소 삭제 |

### 언제 std::deque를 써야 할까?
- 양쪽 끝에서 삽입/삭제가 자주 일어날 때
- 임의 접근(인덱스 접근)이 필요할 때
- 벡터보다 빠른 앞쪽 삽입/삭제가 필요할 때

- 중간 삽입/삭제가 많다면 std::list가 더 나음
- 뒤에서만 추가한다면 std::vector가 더 나음

### 최적화 예제 (대기열 관리 시스템)
```cpp
#include <iostream>
#include <deque>

int main() {
    std::deque<std::string> queue;

    queue.push_back("User1");
    queue.push_back("User2");
    queue.push_back("User3");

    std::cout << "현재 첫 번째 유저: " << queue.front() << std::endl;

    queue.pop_front();  // 첫 번째 유저 제거

    std::cout << "새로운 첫 번째 유저: " << queue.front() << std::endl;

    return 0;
}
```

**출력 값:**
```
현재 첫 번째 유저: User1
새로운 첫 번째 유저: User2
```

## list (이중 연결 리스트)

**특징:** 노드들이 연결된 형태로 저장됨.

**장점:**
- 중간 삽입/삭제 빠름 (O(1))
- 데이터 크기 변경 자유로움

**단점:**
- 랜덤 접근 불가능 (O(N))
- 메모리 사용량 많음 (포인터 추가 저장)

```cpp
#include <iostream>
#include <list>
using namespace std;

int main() {
    list<int> l = {2, 3, 4};
    l.push_front(1);  // 앞에 추가
    l.push_back(5);   // 뒤에 추가

    auto it = l.begin();
    advance(it, 2);   // 두 번째 원소 뒤에 삽입
    l.insert(it, 10);

    cout << "List: ";
    for (int num : l) cout << num << " ";
    cout << endl;
}
```

**언제 사용?**
중간 삽입/삭제가 자주 발생할 때!

## forward_list (단일 연결 리스트)

**특징:** list와 비슷하지만 단방향으로만 연결됨.

**장점:**
- list보다 메모리 절약됨.
- 삽입/삭제 성능 동일 (O(1))

**단점:**
- 뒤에서 접근 불가능 (O(N))
- reverse() 없이는 역순 탐색 불가능

```cpp
#include <iostream>
#include <forward_list>
using namespace std;

int main() {
    forward_list<int> fl = {2, 3, 4};
    fl.push_front(1);  // 앞에 추가

    cout << "Forward List: ";
    for (int num : fl) cout << num << " ";
    cout << endl;
}
```

**언제 사용?**
메모리를 아끼면서 중간 삽입/삭제가 필요할 때!

## array (고정 크기 배열)

**특징:** vector와 유사하지만 크기가 고정됨.

**장점:**
- 일반 배열보다 STL 기능 사용 가능
- 메모리 연속적 → 빠름

**단점:**
- 크기가 고정되어 있음 (std::vector보다 유연성 부족)

```cpp
#include <iostream>
#include <array>
using namespace std;

int main() {
    array<int, 3> arr = {1, 2, 3};

    cout << "Array: ";
    for (int num : arr) cout << num << " ";
    cout << endl;

    cout << "첫 번째 원소: " << arr.front() << endl;
}
```

**언제 사용?**
고정된 크기의 데이터를 다룰 때!

일반 C 배열과 다르게 STL 기능을 지원
벡터처럼 동적 크기가 아니고, 크기가 고정됨
컴파일 타임에 크기가 결정됨 → 런타임 크기 변경 불가능
STL 기능을 지원

```cpp
#include <array>

int main() {
    std::array<int, 5> arr1;  // 크기가 5인 배열 (쓰레기 값 포함)
    std::array<int, 5> arr2 = {1, 2, 3, 4, 5};  // 리스트 초기화
    std::array<int, 5> arr3{10, 20};  // 나머지는 0으로 채워짐 (10, 20, 0, 0, 0)
    
    std::cout << "첫 번째 요소: " << arr2[0] << std::endl;
}
```

크기가 반드시 필요 (vector와 다름)

### 주요 함수들
| 함수 | 설명 |
|------|------|
| `size()` | 배열 크기 반환 |
| `at(index)` | 특정 인덱스 값 반환 (범위 검사 O) |
| `front()` | 첫 번째 요소 반환 |
| `back()` | 마지막 요소 반환 |
| `fill(value)` | 모든 요소를 value로 초기화 |
| `swap(other_array)` | 두 개의 array를 교환 |

```cpp
int main() {
    std::array<int, 5> arr = {10, 20, 30, 40, 50};

    // `operator[]` 사용
    std::cout << arr[2] << std::endl;  // 출력: 30

    // `.at()` 사용 (범위 체크 O)
    std::cout << arr.at(3) << std::endl;  // 출력: 40

    // `.front()`, `.back()`
    std::cout << arr.front() << std::endl;  // 첫 번째 요소 (10)
    std::cout << arr.back() << std::endl;   // 마지막 요소 (50)
}
```

`at()`은 범위를 체크하므로, 벗어나면 std::out_of_range 예외 발생

### 초기화 방법
```cpp
std::array<int, 3> arr1 = {1, 2, 3};  // 직접 초기화
std::array<int, 3> arr2 = {};         // 모든 값 0으로 초기화
std::array<int, 3> arr3 = {0};        // 첫 번째만 0, 나머지는 0
std::array<int, 3> arr4;              // 쓰레기 값 존재
arr4.fill(10);  // 모든 요소를 10으로 채우기
```

```cpp
int main() {
    std::array<int, 5> arr = {1, 2, 3, 4, 5};

    std::cout << "크기: " << arr.size() << std::endl;  // 5

    // `.fill(값)` → 모든 요소를 특정 값으로 채우기
    arr.fill(100);
    
    for (int num : arr) {
        std::cout << num << " ";  // 출력: 100 100 100 100 100
    }
}
```

주의: std::vector처럼 크기를 변경(resize())할 수 없음!

```cpp
std::cout << arr[1] << std::endl;   // 2 (범위 체크 X)
std::cout << arr.at(1) << std::endl; // 2 (범위 체크 O)

std::array<int, 5> arr;
arr.fill(100);  // 모든 요소를 100으로 설정
```

### swap 사용법
```cpp
int main() {
    std::array<int, 5> arr1 = {1, 2, 3, 4, 5};
    std::array<int, 5> arr2 = {10, 20, 30, 40, 50};

    // `.swap()` → 두 배열 교환
    arr1.swap(arr2);

    for (int num : arr1) {
        std::cout << num << " ";  // 출력: 10 20 30 40 50
    }
}
```

주의: swap()은 같은 크기의 std::array에서만 동작 가능

### 반복문 사용
```cpp
int main() {
    std::array<int, 5> arr = {1, 2, 3, 4, 5};

    // 일반 for문
    for (size_t i = 0; i < arr.size(); i++) {
        std::cout << arr[i] << " ";
    }

    // 범위 기반 for문
    for (int num : arr) {
        std::cout << num << " ";
    }

    // 반복자 사용
    for (auto it = arr.begin(); it != arr.end(); it++) {
        std::cout << *it << " ";
    }

    return 0;
}
```

### 메모리 사용량
| 컨테이너 | 메모리 사용량 (추가 오버헤드) |
|----------|---------------------------|
| vector | 적음 (연속된 메모리) |
| deque | 약간 많음 (블록 구조) |
| list | 많음 (노드 + 포인터 추가 필요) |
| forward_list | 많음 (단일 노드 + 포인터 추가) |
| array | 가장 적음 (고정 크기) |

- vector와 array는 연속된 메모리를 사용해서 캐시 효율이 좋음
- list는 포인터(주소) 공간이 추가로 필요

- 앞/중간 삽입이 많으면 list 사용!
- 빠른 임의 접근이 필요하면 vector 사용!

## 연관 컨테이너

키(key)를 기준으로 데이터를 빠르게 검색, 삽입, 삭제할 수 있도록 만들어진 컨테이너들이야. 기본적으로 이진 탐색 트리(Red-Black Tree) 기반으로 동작해.

### std::set 간단 예제
```cpp
#include <iostream>
#include <set>

int main() {
    std::set<int> s = {5, 1, 3, 2};

    s.insert(4);  // 삽입
    s.erase(1);   // 삭제

    for (int val : s) {
        std::cout << val << " ";
    }
    // 출력: 2 3 4 5 (자동 정렬됨)
}
```

### std::map 간단 예제
```cpp
#include <iostream>
#include <map>
#include <string>

int main() {
    std::map<std::string, int> age;
    age["Alice"] = 25;
    age["Bob"] = 30;

    for (const auto& pair : age) {
        std::cout << pair.first << ": " << pair.second << "\n";
    }
    // 출력: Alice: 25
    //       Bob: 30
}
```

### 언제 사용하냐면?
- 중복 없는 데이터가 필요할 때 → set
- 중복 데이터도 허용하고 싶을 때 → multiset
- 키를 기준으로 값을 빠르게 찾고 싶을 때 → map
- 키 중복도 허용하고 싶을 때 → multimap

## set

std::set은 내부적으로 이진 탐색 트리(보통은 Red-Black Tree) 로 구현되어 있어서:
- 원소들은 자동으로 정렬돼 있어
- 중복된 값은 저장되지 않아
- 검색, 삽입, 삭제가 평균적으로 O(log n)이야

### 예제 코드
```cpp
#include <iostream>
#include <set>

int main() {
    std::set<int> s;

    s.insert(3);
    s.insert(1);
    s.insert(4);
    s.insert(1);  // 중복된 값은 무시됨

    for (int val : s) {
        std::cout << val << " ";  // 출력 값: 1 3 4
    }

    return 0;
}
```

삽입 순서와는 상관없이 자동 정렬됨
1을 두 번 넣었지만 하나만 저장됨

### 주요 함수(문법)
| 함수 | 설명 |
|------|------|
| `insert(value)` | 값 삽입 (중복 안됨) |
| `find(value)` | 값 찾기 (있으면 반복자 반환, 없으면 end() 반환) |
| `erase(value)` | 값 삭제 |
| `clear()` | 모든 원소 삭제 |
| `size()` | 현재 원소 개수 |
| `empty()` | 비어있는지 확인 |

```cpp
if (s.find(4) != s.end()) {
    std::cout << "4가 있습니다!";
}
```

이건 약간 헷갈릴 수 있는데 그니까 `s.find(4)`는 4를 반환해 그니까 `auto it = s.find(4);` 이런식으로 출력을 하면 4라는 값이 출력이 되지 근데 이제 4라는 값이 없으면 아마 오류가 걸릴거야 근데 이거를 왜 이야기하냐면 `s.end()`는 비어있는 값이야 근데 4라는 값이 없으면 `s.end()` 값을 반환한단말이지 그러면 4라는 값이 없을 때 if문에서는 `if(s.end() != s.end())` 이런식으로 되다 보니까 거짓이 나와 그래서 4가 있으면 `4 != s.end()` 이래서 참이 되는거고 그래서 그런거야

### 정렬 기준 변경하기 (부등호 반대로)
```cpp
#include <set>
#include <functional>  // greater 사용을 위해

std::set<int, std::greater<int>> s;  // 내림차순 정렬
```

## multiset

set은 중복을 허용하지 않지만, multiset은 같은 값 여러 개 저장 가능해!

### 특징
- **정렬됨**: 자동으로 오름차순 정렬됨 (set과 동일)
- **중복 허용**: 같은 값을 여러 개 저장할 수 있음
- **삽입/삭제 빠름**: 내부적으로 균형 이진 탐색 트리로 관리됨 (보통 Red-Black Tree)
- **반복자 지원**: begin(), end(), find(), erase() 등 사용 가능

### 예제 코드
```cpp
#include <iostream>
#include <set>

int main() {
    std::multiset<int> ms;

    ms.insert(3);
    ms.insert(1);
    ms.insert(3);
    ms.insert(2);
    ms.insert(1);

    for (int num : ms) {
        std::cout << num << " ";
    }
    // 출력: 1 1 2 3 3
}
```

정렬되어 있고, 중복이 들어감!

### 주요 함수
| 함수 | 설명 |
|------|------|
| `insert(val)` | 값 삽입 |
| `find(val)` | 해당 값 중 하나를 가리키는 반복자 반환 |
| `count(val)` | 해당 값이 몇 개 있는지 반환 |
| `erase(val)` | 해당 값을 모두 삭제 |
| `erase(iterator)` | 해당 위치의 하나만 삭제 |

### count() 사용법
```cpp
std::multiset<int> ms = {1, 2, 2, 2, 3};
std::cout << ms.count(2);  // 출력: 3
```

### find()는 하나만 가리킴
```cpp
auto it = ms.find(2);  // 여러 개 중 하나만 가리켜
```

### 여러 개 지우기: equal_range()
```cpp
auto range = ms.equal_range(2);
ms.erase(range.first, range.second);  // 2 전부 삭제
```
