# 모던 C++ 기본 문법

## auto

쉽게 말해서 값을 미리 초기화를 하면 그 값 대로 타입(자료형)으로 된다.

```cpp
auto a = 10;       // a = int로 추론됨
auto b = 3.14;    // b = double로 추론됨
auto str = "Hi";   // str은 const char*로 추론됨
```

### 예시
```cpp
auto add(int a, int b) {
    return a + b; // 컴파일러가 int로 추론
}

int main() {
    cout << add(3, 5) << endl; // 8
    return 0;
}
```

이거 쓰면 코드가 간결해지고
템플릿과 함께 사용하기 좋음
복잡한 타입 (ex: iterator)도 쉽게 선언 가능

```cpp
#include <vector>     // 라이브러리

std::vector<int> vec = { 1, 2, 3, 4, 5 };   // 정수형 벡터를 선언하고 값을 초기화

// 기존 방식
std::vector<int>::iterator it = vec.begin();

// auto 사용
auto it2 = vec.begin();
```

### 설명!
auto는 자동으로 타입을 추론하는 타입이야 위에 코드를 보면 알겠지만 자동으로 타입(자료형)이나 원소 같은 값을 받고 추론하는걸 알 수 있어 대신에 그냥 선언은 안돼 그니까 예시로 들자면

```cpp
auto a;   // 이렇게는 안돼 왜냐면 값을 받고 추론을 해야하는데 이건 그냥 선언만 한거라서 이렇게 되면 오류가 돼 그래서 변수를 생성하고 바로 값을 받아야해
auto a = 1;  // int
```

이런식으로

C++에서는 배열(array)을 대신할 수 있는 동적 크기 변경이 가능한 컨테이너로 vector를 제공해 vector는 C++ 표준 라이브러리의 일부라서 `<vector>` 헤더를 포함해야해

`std::vector<int> vec = { 1, 2, 3, 4, 5 };` 정수형 벡터를 선언하고 값을 초기화를 함
근데 이 벡터는 배열과 비슷하지만 크기를 자동으로 조정할 수 있음
새로운 요소를 추가하거나 삭제할 수 있음

C언어랑 파이썬 언어에 차이점중에서 배열과 리스트가 있는데 배열은 크기를 한번 선언하면 못바꾸지만 파이썬에 있는 리스트는 값을 추가하고 삭제 할 수 있음 근데 C++에 있는 벡터는 리스트처럼 값을 추가하고 삭제할 수 있음

### 예시
```cpp
int arr[] = { 1, 2, 3, 4, 5 };     //크기 변경 불가능
std::vector<int> vec = { 1, 2, 3, 4, 5 };    // 크기 변경 가능
vec.push_back(6);       // 새로운 요소 추가 기능
```

### 다음 std::vector<int>::iterator it = vec.begin(); 이 코드 줄에 대해서 설명하자면
`vec.begin();` 벡터의 첫 번째 원소를 가리키는 반복자를 반환함 즉 쉽게 말해서 vec[0] 벡터에 0번째 원소(반복자)를 준거나 마찬가지

### 원소란?
- 컨테이너(벡터, 배열 등)에 저장된 각각의 값을 의미
- `std::vector<int> vec = {1, 2, 3, 4, 5};`
- vec의 원소들은 1 2 3 4 5
- vec[0] = 1 (첫 번째 원소)
- vec[1] = 2 (두 번째 원소)
- vec[2] = 3 (세 번째 원소)
- 벡터 내부에 저장된 값 하나하나가 원소

```cpp
std::cout << vec[2];   // 3 출력
```

### 반복자란?
- 반복자는 포인터처럼 동작하지만 포인터 자체는 아님
- 벡터 내부에서 원소들이 연속된 메모리 공간에 저장되므로 배열처럼 동작함

그리고 iterator를 선언할 때는
`std::vector<int>::iterator it;`  // 벡터<int>의 반복자 선언
이런식으로 선언해야 함 기본적인 선언 방식

그리고 포인터랑 비슷하게 동작을해서 반드시 무언가를 가리켜야 함
`std::vector<int>::iterator it = 1;`  // 에러 발생
벡터 내부의 원소를 가리켜야 함

```cpp
int it2 = vec.begin();  // 오류
```

이런식으로 변수를 생성하면 오류! 이유는?
`vec.begin()`이 반환하는 것은 `std::vector<int>::iterator` 이걸 반환하는거야
근데 int는 iterator 타입이 아니므로 오류 발생

iterator 이거는 포인터는 아니지만 포인터처럼 동작하는 객체라고 생각하면 돼
내부적으로는 원소를 가리키고 있지만 클래스 객체이기 때문에 다양한 연산을 제공함

```cpp
// 기존 방식
vector<int>::iterator it = vec.begin();
cout << *it << endl; // 1

// auto 사용
auto it2 = vec.begin();
cout << *it2 << endl; // 1
```

```cpp
std::vector<int>::iterator it;   // 명시적으로 반복자 선언
auto it2 = vec.begin();         // 자동으로 반복자 타입 추론
```

auto를 쓰면 이런식으로 코드가 간결해짐

그리고 begin()이 있으면 end도 있음 `vec.end()`
end()는 마지막 원소 다음을 가리키는 반복자야

### 예시
```cpp
std::vector<int> vec = { 1, 2, 3, 4, 5 };
for (auto it = vec.begin(); it != vec.end(); ++it) {
    cout << *it << " ";
}
```

**출력 값:** 1 2 3 4 5

즉 여기서 `vec.end()`는 `vec[5]`(존재하지 않는 공간)을 가리켜
필요한 이유는 반복문에서 begin()과 end()가 이전까지 순회하려면 필요해
end()는 실제 원소가 아니라 순회를 멈출 위치를 제공하는거라고 생각하면 돼

### 예시
```cpp
#include <iostream>
#include <vector>
#include <iterator>  // std::advance()를 사용하려면 필요

int main() {
    std::vector<int> vec = {10, 20, 30, 40, 50};

    auto it = vec.begin();  // 첫 번째 원소
    std::advance(it, 2);  // 2칸 이동 (30을 가리킴)  advance 반복자를 n칸 이동

    std::cout << *it;  // 30 출력
}
```

```cpp
int main() {
    const int num = 42;
    
    auto a = num;           // int (const 제거됨)
    const auto b = num;  // const 유지
    auto& c = num;        // const 유지 (참조)
    
    cout << a << " " << b << " " << c << endl;
    
    return 0;
}
```

### 설명
num은 const인데 `auto a = num;` 여기 코드줄은 auto가 const가 아니기 때문에 const로 그니까 상수로 변수가 안됨 근데 const auto나 auto로 &(참조)참조를 하고 있는 b와 c는 const가 그대로 유지가 됨

```cpp
std::vector<int> vec = {1, 2, 4};
auto it = vec.begin() + 2;  // 인덱스 2 위치 (4 앞) 인덱스 2 위치 원소 값을 받음 포인터 주소값처럼
vec.insert(it, 3);  // {1, 2, 3, 4}
//insert()로 특정 위치에 값을 추가 가능
```

```cpp
vec.emplace_back(5);  // {1, 2, 3, 4, 5} 객체 생성과 함께 추가
```

emplace_back()은 std::bector에서 사용하는 함수로 벡터의 마지막에 원소를 추가하는 함수입니다.
그런데 push_back()과 다르게 원소를 추가하는 동시에 해당 원소를 생성하는 방식으로 동작

1. push_back()이건 기존에 생성된 객체를 벡터에 추가하는 방식
2. emplace_back()이거는 객체를 생성하면서 벡터의 마지막에 그 객체를 추가하는 방식입니다. 즉 객체 생성과 추가가 한 번에 일어납니다

### 예시
```cpp
std::vector<int> vec = {1, 2, 3, 4};
vec.emplace_back(5);  // {1, 2, 3, 4, 5} 객체 생성과 함께 추가
//생성자를 호출화면서 원소를 추가하므로 객체가 이미 생성되어 있는 5를 추가하는 push_back()랑은 차이가 있음 
```

벡터의 끝에 추가하는 함수

```cpp
std::vector<int> vec;
//std::vector<int> vec = {1, 2, 3, 4};
vec.emplace_back(5);   // vec = { 5 }
```

이렇게 되면 벡터가 생성되면서 5 값이 들어감

```cpp
std::vector<int> vec;
vec.emplace_back(5);  // 벡터에 5 추가
vec.emplace_back(6);  // 벡터에 6 추가
// vec = { 5, 6 }
```

```cpp
std::vector<int> vec;
int x = 5;
vec.push_back(x);  // x가 이미 존재하는 객체라면, 그 객체를 추가
// vec = { 5 }
```

## decltype

```cpp
int add(int a, int b) { 
     return a + b;
}

int main() {
    decltype(add(1, 2)) result;  // add 함수의 리턴 타입(int)을 가져옴
    result = add(3, 4);
    std::cout << result << std::endl;  // 7 출력
}
```

decltype은 이미 선언된 변수/표현식에 타입을 가져옴

`decltype(add(1, 2)) result;` 여기 부분에서 result는 값을 받는게 아니라 타입만 가져온다고 생각하면 됨
그러면 add 함수에 1하고 2에 값이 가냐? 라고 물어보면 정확히는 가지는 않아 타입 추론을 위해 쓰는 것뿐 그래서 값은 안가고 int 타입만 반환해
그러면 `decltype(add()) result;` 이렇게 해도 되는거 아니야? 라고 물어볼 수 있지만 이러면 오류가 걸려 왜냐면 add 함수에 값을 받아야하는게 2개나 있기 때문에 오류가 걸려
그래서 사실상 `decltype(add(1, 2)) result;` 여기서 add 값은 아무 값이나 넣어도 괜찮아
