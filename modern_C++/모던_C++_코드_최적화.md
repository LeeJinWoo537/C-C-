# 모던 C++ 코드 최적화

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

값이 있으면 42를 출력하는 데

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
