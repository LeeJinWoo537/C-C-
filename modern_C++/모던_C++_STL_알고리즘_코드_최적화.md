# 모던 C++ STL 알고리즘, 코드 최적화

## constexpr

### 컴파일 계산
- 프로그램이 컴파일될 때 발생하는 시간
- 코드를 기계어로 번역하는 과정에서 일어나는 시간
- 컴파일러가 코드의 문법을 확인하고 오류가 있으면 이때 잡아냄

### 예시 코드
```cpp
constexpr int square(int x) {
    return x * x;
}

int main() {
    int result = square(5);  // 컴파일 타임에 5 * 5 = 25로 미리 계산됨
    std::cout << result << std::endl;  // 출력: 25
    return 0;
}
```

`square(5)`는 컴파일될 때 25로 바뀌어서 코드에 직접 들어감
실행할 때 `square(5)`를 계산할 필요가 없음 → 최적화

### 런타임 값과의 차이
```cpp
int runtime_value() {
    return 42;
}

int main() {
    constexpr int a = 10;  // OK: 컴파일 타임에 결정됨
    int b = runtime_value();
    // constexpr int c = b;  // 오류: b는 런타임 값이라 constexpr 불가
}
```

### 클래스에서의 constexpr 사용
```cpp
class Math {
public:
    constexpr int add(int x, int y) const {
        return x + y;
    }
};

int main() {
    constexpr Math math;  // constexpr 객체
    constexpr int sum = math.add(3, 4);  // 7, 컴파일 타임 계산
    std::cout << sum << std::endl;
}
```

### 상수 표현이 가능 (배열 크기 등)
- 배열 크기는 컴파일 타임에 정해져야 함
- constexpr을 사용하면 컴파일 타임에 배열 크기를 설정할 수 있음

```cpp
constexpr int SIZE = 10;
int arr[SIZE];  // 배열 크기가 컴파일 타임에 정해짐
```

- SIZE가 constexpr이라서 배열 크기가 컴파일 타임에 결정됨
- 만약 `const int SIZE = 10;` 이렇게 하면 컴파일러가 최적화해주긴 하지만, constexpr만큼 강력하지 않음!

### 컴파일 타임에 안정성 체크 가능
컴파일 타임에 값이 결정되기 때문에 잘못된 값이 들어가면 컴파일 에러가 발생함
즉, 실행하기 전에 오류를 잡을 수 있음!

```cpp
constexpr int get_value(int x) {
    return x > 0 ? x : throw "x must be positive!";
}

int main() {
    constexpr int val = get_value(-5);  // 컴파일 에러 발생!
    return 0;
}
```

컴파일 타임에 오류가 잡히기 때문에 실행 자체가 안 됨!

그래서 constexpr은 계산을 하는 변수나 함수를 사용할 때 좋음 상수(const)대신에 사용하는게 좋음

const는 약간 변하지 않은 값 `const PI = 3.14;` 약간 이런거 할 때 사용하기 좋음

## std::move

객체의 소유권을 이전할 때 사용
- 객체를 복사하는 대신 "이동" 시켜서 불필요한 복사를 줄이고 최적화함.

### 예시 코드
```cpp
#include <iostream>
#include <vector>

int main() {
    std::vector<int> v1 = {1, 2, 3};
    std::vector<int> v2 = std::move(v1);

    std::cout << "v1 size: " << v1.size() << std::endl;  // 0 (이동됨)
    std::cout << "v2 size: " << v2.size() << std::endl;  // 3

    return 0;
}
```

### 비효율적인 복사
```cpp
#include <iostream>
#include <vector>

std::vector<int> createVector() {
    std::vector<int> vec = {1, 2, 3, 4, 5};
    return vec;  // ⛔ 복사 발생 (비효율적)
}

int main() {
    std::vector<int> myVec = createVector();  // 복사가 일어남
    return 0;
}
```

복사본이 생겨서 불필요한 메모리 사용 증가

### 효율적인 이동
```cpp
#include <iostream>
#include <vector>

std::vector<int> createVector() {
    std::vector<int> vec = {1, 2, 3, 4, 5};
    return std::move(vec);  //  이동 (복사 없이 소유권 이전)
}

int main() {
    std::vector<int> myVec = createVector();  // 이동 연산자 호출 (빠름!)
    return 0;
}
```

`std::move(vec)`를 사용하면 벡터의 소유권이 이전됨 → 복사 없이 이동!
성능 향상! 불필요한 메모리 낭비 방지!

### 클래스에서의 이동 생성자
```cpp
#include <iostream>
#include <vector>

class Data {
public:
    std::vector<int> values;

    Data(std::vector<int> v) : values(std::move(v)) {}  // 이동 생성자
};

int main() {
    std::vector<int> vec = {1, 2, 3, 4};
    Data d(std::move(vec));  // vec의 데이터를 d로 이동

    std::cout << "vec size: " << vec.size() << std::endl;  // 0 (비어 있음)
}
```

### 사용할 때 주의할 점
```cpp
std::string str = "Hello";
std::string newStr = std::move(str);  // str의 소유권이 newStr로 이동됨

std::cout << str;  // ⚠️ str은 비어있을 수도 있음 (사용 주의)
//std::move를 사용하면 원본 객체를 더 이상 보장할 수 없음!
```

## std::optional

값이 없을 수도 있는 변수를 다룰 때 사용하는 타입

### 예시 코드
```cpp
#include <iostream>
#include <optional>

std::optional<int> getValue(bool giveValue) {
    if (giveValue)
        return 42;
    return std::nullopt;  // 값이 없음
}

int main() {
    std::optional<int> val = getValue(true);
    if (val) {
        std::cout << "Value: " << *val << std::endl;  // 42
    } else {
        std::cout << "No value" << std::endl;
    }
}
```

`std::nullopt`을 반환하면 값이 없는 상태가 됨
값이 있으면 `*val`로 접근할 수 있음

### value_or() 활용
값이 없을 경우 기본 값을 제공할 수 있음

```cpp
std::cout << val.value_or(100) << std::endl;  // 값이 없으면 100 출력
```

이게 어떻게 활용이 되냐면

```cpp
int main() {
    std::optional<int> val = 42;  // 값이 있음

    std::cout << val.value_or(100) << std::endl;  // 42 출력

    return 0;
}
```

값이 있으면 42를 출력하는데

값이 없으면
```cpp
int main() {
    std::optional<int> val = std::nullopt;  // 값이 없음

    std::cout << val.value_or(100) << std::endl;  // 100 출력

    return 0;
}
```

값이 없으면 100을 출력!

왜 이렇게 하냐?라고 한다면
```cpp
std::cout << val.value() << std::endl;
```

이상태에서 값이 없는 상태에서 출력을 하면
val이 `std::nullopt`이면 런타임 에러 발생!
`value_or(기본값)`을 쓰면 안전하게 기본값을 설정할 수 있음

### has_value() 사용
```cpp
#include <iostream>
#include <optional>

std::optional<int> getValue(bool giveValue) {
    if (giveValue)
        return 42; // 값이 있을 때
    else
        return std::nullopt; // 값이 없을 때
}

int main() {
    std::optional<int> result = getValue(true);

    if (result.has_value()) {
        std::cout << "Value: " << result.value() << std::endl;  // 42 출력
    } else {
        std::cout << "No value!" << std::endl;
    }

    return 0;
}
```

- `std::optional<int>`: int 값을 가질 수도 있고 없을 수도 있음
- `std::nullopt`: 값이 없음을 의미
- `has_value()`: 값이 있는지 확인
- `value()`: 실제 값을 가져옴 (값이 없으면 예외 발생)

### 함수에서 오류 처리
```cpp
#include <iostream>
#include <optional>

std::optional<std::string> findUserById(int id) {
    if (id == 1) return "Alice";
    else return std::nullopt; // 사용자를 찾지 못한 경우
}

int main() {
    std::optional<std::string> user = findUserById(2);

    if (user) {
        std::cout << "User found: " << *user << std::endl;  // *user는 value()와 동일
    } else {
        std::cout << "User not found" << std::endl;
    }

    return 0;
}
```

그리고 위에 있는 함수에서
```cpp
std::optional<std::string> findUserById(int id) {
    if (id == 1) return "Alice";
    else return std::nullopt; // 사용자를 찾지 못한 경우
}
```

`<>` 여기 안에 있는 타입은 string인데 함수에서 받는 값은 int 타입이니까 오류가 나는거 아니냐? 라고 할 수 있지만 `< >`여기 들어간 타입은 반환 타입만 신경 쓰면 된다 return에서 반환할 타입만 신경 쓰면 된다 그래서 함수에서 받는 타입은 아무 타입이나 받아도 상관 없다.

### 출력 방법
출력을 할 때는
```cpp
std::cout << opt.value();  //  OK!
std::cout << *opt;  //  OK!
```

이렇게 출력을 해야함
둘의 차이점이라고 하면
- `.value()` → 값이 없으면 예외 발생
- `*` (역참조) → 값이 없으면 UB(정의되지 않은 동작, 위험)

## 문제 해결

### 문제1
```cpp
#include <iostream>

int factorial(int n) {
    if (n <= 1) return 1;
    return n * factorial(n - 1);
}

int main() {
    constexpr int result = factorial(5);
    std::cout << "Factorial of 5: " << result << std::endl;
    return 0;
}
```

컴파일 타임이 미리 계산되도록 변경

**답안:**
```cpp
#include <iostream>

constexpr int factorial(int n) {
    if (n <= 1) return 1;
    return n * factorial(n - 1);
}

int main() {
    constexpr int result = factorial(5);
    std::cout << "Factorial of 5: " << result << std::endl;
    return 0;
}
```

### 문제2
```cpp
#include <iostream>
#include <string>

std::string getMessage() {
    std::string msg = "Hello, world!";
    return msg;
}

int main() {
    std::string message = getMessage();
    std::cout << message << std::endl;
    return 0;
}
```

move를 적절히 사용해서 문자열 복사를 줄여라

**틀린 답안:**
```cpp
#include <iostream>
#include <string>

std::string getMessage() {
    std::string msg = "Hello, world!";
    return std::move(msg);
}

int main() {
    std::string message = getMessage();
    std::cout << message << std::endl;
    return 0;
}
```

**틀린 이유:**
`return std::move(msg);` 여기 부분에서 굳이 move를 할 필요가 없음
msg는 함수가 끝날 때 RVO(Return Value Optimization)로 최적화될 수 있기 때문이야.
최신 컴파일러에서는 std::move 없이도 RVO로 최적화되기 때문에 굳이 필요하지 않아!
결론: std::move는 객체가 명확히 더 이상 필요하지 않을 때만 사용해야 해!

`return "Hello world!";`     // 이런식으로 해도 RVO가 적용됨

**개선된 코드:**
```cpp
#include <iostream>
#include <string>

std::string createMessage() {
    std::string msg = "Hello, world!";
    return msg;  // RVO 최적화 자동 적용 (std::move 필요 없음)
}

int main() {
    std::string message = std::move(createMessage());  // 이동 연산자로 받음
    std::cout << message << std::endl;
    return 0;
}
```

여기서 RVO는 뭐냐면 C++ 컴파일러가 불필요한 복사를 줄이기 위해 자동으로 적용하는 최적화 기법이야.

- 일반적으로 함수가 객체를 반환하면 복사 연산자 또는 이동 연산자가 호출됨.
- 하지만 컴파일러가 최적화를 수행해서 "복사 자체를 없애버림"
- 즉, 반환값을 직접 호출한 쪽의 변수에 바로 생성하는 방식으로 동작해!

- 보통 `return msg;` 하면, msg가 함수 내부의 지역 변수라서 사라져야 함.
- 그런데 컴파일러가 최적화해서 message 변수에 직접 값을 생성함!
- 그래서 복사 연산자나 이동 연산자가 호출되지 않음 → 성능 최적화!

`std::string message = std::move(createMessage());`

여기서 `std::move()`를 쓰면, 함수 반환값을 이동 시멘틱(move semantics)으로 받겠다는 뜻이야.
즉, RVO가 적용되지 않는 경우에도 이동 연산자를 강제로 사용하도록 유도할 수 있음!

### 이동 시멘틱이 뭐냐면
- C++11부터 도입된 개념으로, 객체의 복사가 아니라 "이동"을 수행하는 방식이야.

### std::move()를 사용해야 하는 경우
- RVO가 강제되지 않는 경우
- 예를 들어, 다양한 경로에서 다른 변수를 반환하는 경우
- std::vector, std::map 같은 큰 컨테이너를 반환할 때

```cpp
std::string createMessage(bool flag) {
    std::string msg1 = "Hello";
    std::string msg2 = "World";
    
    if (flag)
        return msg1;  // RVO 적용 안 됨!
    else
        return msg2;  // RVO 적용 안 됨!
}

int main() {
    std::string message = std::move(createMessage(true));  // 이동 연산자로 받기
    std::cout << message << std::endl;
    return 0;
}
```

```cpp
// 복사 (비효율적)
std::vector<int> v2 = v1; // v1의 데이터를 새로운 메모리에 복사함

// 이동 (효율적)
std::vector<int> v3 = std::move(v1); // v1의 데이터를 v3로 "이동"시키고 v1은 비워짐
```

### 문제3
```cpp
#include <iostream>

int getNumber(bool valid) {
    if (valid)
        return 42;
    else
        return -1;  // 유효하지 않은 값 처리
}

int main() {
    int number = getNumber(false);
    std::cout << "Number: " << number << std::endl;
    return 0;
}
```

std::optional을 사용하여 getNumber 함수가 유효한 값이 없을 때 std::nullopt을 반환하도록 수정하세요.

**틀린 답안:**
```cpp
#include <iostream>
#include <optional>

using namespace std;

operator<int> int getNumber(bool valid) {
    if (valid)
        return 42;
    else
        return nullptr;  // 유효하지 않은 값 처리
}

int main() {
    int number = getNumber(false);

    if (number.has_value()) {
        cout << "Number: " << number.value() << std::endl;
    } 
        
    else {
        cout << "값이 없습니다 ㅠㅠ" << endl;
    }
    
    return 0;
}
```

**틀린 이유:**
- `operator<int>` 여기 부분에서 정답은 `std::operator<int>` 이렇게 작성해야 함
- `return nullptr;` → `std::nullopt`를 사용해야 해!
- main에서 `int number = getNumber(false);` → `std::optional<int>`로 받아야 해

**개선된 코드:**
```cpp
#include <iostream>
#include <optional>

using namespace std;

std::optional<int> getNumber(bool valid) {  // std::optional<int> 반환
    if (valid)
        return 42;
    else
        return std::nullopt;  // 값이 없을 때 nullopt 반환
}

int main() {
    std::optional<int> number = getNumber(false);  // optional로 받아야 함!

    if (number.has_value()) {  // 값이 있으면 출력
        cout << "Number: " << number.value() << endl;
    } else {
        cout << "값이 없습니다 ㅠㅠ" << endl;
    }
    
    return 0;
}
```

여기서 아니? 왜? using namespace를 썼는데 왜 std:: 이거를 붙여?? 라고 궁금할 수 있는데 이유는 C++에서는 함수와 변수의 이름이 optional이라는 일반적인 단어와 충돌할 가능성이 있어.
특히, optional이라는 변수를 선언하면 `std::optional<int>`와 혼동될 수 있어!

`optional<int>`가 변수 optional을 사용하려고 시도하는 것처럼 보일 수 있음
그래서 컴파일러가 `optional<int>`가 `std::optional<int>`인지, 그냥 `optional<int>`인지 헷갈릴 수 있음
이런 문제를 막기 위해 `std::optional<int>`를 명확하게 써주는 게 좋아!

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

std::list는 이중 연결 리스트 (Doubly Linked List) 를 구현한 C++ STL 컨테이너야.
배열 기반 컨테이너(vector, deque)와 다르게, 요소들이 연속된 메모리 공간에 저장되지 않고 각 노드가 다음 및 이전 노드의 주소를 가짐.

### list의 특징
- 빠른 삽입 및 삭제: 중간에 요소를 삽입하거나 삭제할 때 O(1)
- 임의 접근이 불가능: operator[] 사용 불가. vector처럼 빠르게 특정 인덱스에 접근할 수 없음
- 연속된 메모리 할당 X: 요소가 각각의 노드로 존재하여 메모리 사용량이 상대적으로 높음

### list 선언 및 기본 사용법
```cpp
#include <iostream>
#include <list>

int main() {
    std::list<int> myList = {1, 2, 3, 4, 5};

    // 요소 추가
    myList.push_back(6); // 뒤에 추가
    myList.push_front(0); // 앞에 추가

    // 요소 삭제
    myList.pop_back(); // 뒤 요소 삭제
    myList.pop_front(); // 앞 요소 삭제

    // 순회
    for (int num : myList) {
        std::cout << num << " ";
    }

    return 0;
}
```

### 주요 함수들
| 함수 | 설명 |
|------|------|
| `push_back(val)` | 맨 뒤에 요소 추가 |
| `push_front(val)` | 맨 앞에 요소 추가 |
| `pop_back()` | 맨 뒤 요소 삭제 |
| `pop_front()` | 맨 앞 요소 삭제 |
| `insert(it, val)` | 특정 위치 it에 요소 추가 |
| `erase(it)` | 특정 위치 it의 요소 삭제 |
| `clear()` | 모든 요소 제거 |
| `size()` | 현재 요소 개수 반환 |
| `empty()` | 비어 있는지 확인 |
| `front()` | 첫 번째 요소 반환 |
| `back()` | 마지막 요소 반환 |

### 중간 삽입 및 삭제 (iterator 사용)
std::list는 임의 접근이 불가능하기 때문에 iterator를 사용해서 원하는 위치에 삽입/삭제 가능해.

```cpp
#include <iostream>
#include <list>

int main() {
    std::list<int> myList = {10, 20, 30, 40};

    // 중간 삽입 (30 앞에 25 추가)
    auto it = myList.begin();
    std::advance(it, 2);  // 두 번째 요소(30)로 이동
    myList.insert(it, 25);

    // 중간 삭제 (40 삭제)
    it = myList.end();
    std::advance(it, -1); // 마지막 요소로 이동
    myList.erase(it);

    // 출력
    for (int num : myList) {
        std::cout << num << " ";  // 출력 값: 10, 20, 25, 30
    }
}
```

여기서 advance() 이게 뭐냐고 물어볼 수 있는데 이건 내가 반환하고자 하는 위치를 찾는 역할을 해 그니까 쉽게 말해서 list는 벡터나 배열처럼 `it += 2` 이런식으로 그니까 배열 처럼 `*(it + 2);` 이런식으로 인덱스를 찾는게 안돼 불가능하지 그래서 advence()를 이용해야해서 이걸 사용해서 인덱스 위치에 있는 값을 반환하는거지

`*(it + 2) == advence(it, 2)`이거랑 같아 참고로 백터는 `it += 2` 이렇게 가능함

`myList.begin();`을 사용하는 이유는 시작 주소에서 값을 이동하면서 찾는게 쉬우니까 쓰는거야 굳이 시작 주소나 마지막 주소에서 안써도 상관은 없음

그리고 `it = myList.end();` 이건 알겠지만 list에 마지막 주소 그니까 값을 나타내는게 아니라 값 뒤에 있는 비어있는 무언가를 가리키는거야 begin()은 시작 주소 예를 들어서
`{10, 20, 30, 40}` 이렇게 있으면 begin()은 10을 나타내는데 end()는 40을 나타내는게 아니라 뒤에 비어있는 무언가를 나타내 그니까
`{10, 20, 30, 40, (비어있는 무언가)}` 이런식으로 가리켜

그리고 `advance(it, -1);` 여기에서 -1은 뒤에서 한칸이라는 뜻인데 그니까 쉽게 말해서 
`{10, 20, 30, 40, (비어있는 무언가)};`
`(it, -1)`이면
`(40 <-- 비어있는 무언가)` 이런식으로 `(it, -1)` 30 <-- 40 이런식으로 뒤에서 한칸으로 가는거야

그 외에도
`std::next()`와 `std::prev()` 이 2개가 있는데

```cpp
#include <iterator>  // std::next 필요

int main() {
    std::list<int> myList = {10, 20, 30, 40};

    auto it = myList.begin();  // 첫 번째 요소 (10)
    auto nextIt = std::next(it, 2);  // 2칸 앞으로 이동 (30을 가리킴)

    std::cout << *nextIt << std::endl;  // 출력: 30
}
```

```cpp
#include <iterator>  // std::prev 필요

int main() {
    std::list<int> myList = {10, 20, 30, 40};

    auto it = myList.end();   // end()는 마지막 요소 뒤를 가리킴
    auto prevIt = std::prev(it, 1);  // 1칸 뒤로 이동 (40을 가리킴)

    std::cout << *prevIt << std::endl;  // 출력: 40
}
```

이런식으로 사용

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

std::forward_list는 단일 연결 리스트(Singly Linked List) 를 구현한 STL 컨테이너야.
단일 연결 리스트는 한 방향으로만 탐색 가능!
std::list보다 메모리를 적게 사용하고, 삽입/삭제가 빠름
임의 접근(랜덤 접근) 불가능 → vector[i] ❌, std::next() 사용해야 함

```cpp
#include <iostream>
#include <forward_list>

int main() {
    std::forward_list<int> flist = {10, 20, 30};

    for (int num : flist) {
        std::cout << num << " ";  // 출력: 10 20 30
    }
}
```

std::list와 다르게 size() 함수가 없음 → 직접 개수를 세야 함.

### 요소 삽입 (push_front(), insert_after())
```cpp
#include <iostream>
#include <forward_list>

int main() {
    std::forward_list<int> flist = {20, 30};

    flist.push_front(10);  // 맨 앞에 삽입
    auto it = flist.begin();
    flist.insert_after(it, 15);  // 10 다음에 15 삽입

    for (int num : flist) {
        std::cout << num << " ";  // 출력: 10 15 20 30
    }
}
```

- `insert_after(it, 값)` → "it" 다음에 값 삽입
- `push_front(값)` → 맨 앞에 값 추가

### 요소 삭제 (pop_front(), erase_after(), remove())
```cpp
#include <iostream>
#include <forward_list>

int main() {
    std::forward_list<int> flist = {10, 20, 30, 40};

    flist.pop_front();  // 첫 번째 요소(10) 삭제

    auto it = flist.begin();
    flist.erase_after(it);  // 20 다음 요소(30) 삭제

    flist.remove(40);  // 값이 40인 요소 삭제

    for (int num : flist) {
        std::cout << num << " ";  // 출력: 20
    }
}
```

- `pop_front()` → 첫 번째 요소 삭제
- `erase_after(it)` → "it" 다음 요소 삭제
- `remove(값)` → 특정 값 삭제

### std::next() 활용 (임의 접근 불가)
```cpp
#include <iostream>
#include <forward_list>
#include <iterator>  // std::next() 필요

int main() {
    std::forward_list<int> flist = {10, 20, 30, 40};

    auto it = std::next(flist.begin(), 2);  // 2칸 이동 (30을 가리킴)
    std::cout << *it << std::endl;  // 출력: 30
}
```

`flist[2]`처럼 인덱스로 접근 불가능 → std::next()로 이동해야 함.

### 주요 함수들
| 함수 | 설명 |
|------|------|
| `push_front(값)` | 맨 앞에 요소 추가 |
| `insert_after(반복자, 값)` | 반복자 다음에 요소 추가 |
| `pop_front()` | 첫 번째 요소 삭제 |
| `erase_after(반복자)` | 반복자 다음 요소 삭제 |
| `remove(값)` | 특정 값 삭제 |
| `remove_if(조건)` | 조건을 만족하는 요소 삭제 |
| `front()` | 첫 번째 요소 반환 |
| `clear()` | 모든 요소 삭제 |
| `empty()` | 리스트가 비어 있는지 확인 |
| `assign(개수, 값)` | 리스트를 새 값으로 채우기 |
| `reverse()` | 리스트 순서 반전 |
| `sort()` | 오름차순 정렬 |

### insert_after 사용법
- `insert_after(반복자, 값)` → 반복자 다음에 삽입
  - `auto it = flist.before_begin();`  // 맨 앞 이전 위치
  - `flist.insert_after(it, 15);`  // 15 삽입 (맨 앞)

- `insert_after(반복자, 개수, 값)` → 여러 개 삽입
  - `flist.insert_after(it, 3, 99);`  // 99를 3번 삽입

- `insert_after(반복자, {값1, 값2, 값3})` → 여러 개 삽입
  - `flist.insert_after(it, {10, 20, 30});`

### erase_after 사용법
- `erase_after(반복자)` → 반복자 다음 요소 삭제
  - `auto it = flist.before_begin();`
  - `flist.erase_after(it);`  // 첫 번째 요소 삭제

- `remove(값)` → 해당 값 전체 삭제
  - `flist.remove(10);`  // 값이 10인 모든 요소 삭제

- `remove_if(조건)` → 조건을 만족하는 요소 삭제
  - `flist.remove_if([](int x) { return x % 2 == 0; });`  // 짝수 제거

### std::next 사용법
- `std::next(반복자, n)` → n칸 이동
  - `auto it = std::next(flist.begin(), 2);`  // 2칸 이동
  - `std::cout << *it;`

### assign 사용법
- `assign(개수, 값)` → 새로운 값으로 채우기
  - `flist.assign(5, 100);`  // 100을 5개 넣음

- `assign({값1, 값2, 값3})` → 리스트 값 변경
  - `flist.assign({1, 2, 3});`

### 정렬과 반전
```cpp
flist.sort(std::greater<int>());  // 내림차순 정렬
flist.reverse();  // 요소 순서 반전
```

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
