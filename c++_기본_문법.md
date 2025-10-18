# C++ 기본 문법

## 개요
C++ 프로그래밍의 기본적인 문법과 개념을 설명합니다.

## 기본 출력과 입력

### Hello World 예제
```cpp
#include <iostream>

using namespace std;

int main()
{
    cout << "안녕하세요!" << endl;

    return 0;
}
```

### 입출력 연산자
- `cout <<` == `printf` (출력)
- `cin >>` == `scanf` (입력)
- `endl` == `\n` (줄바꿈)

### 데이터 흐름 방향
`<<` 데이터의 흐름 방향

```cpp
cout << "안녕하세요!";   // 안녕하세요라는 문자를 cout 너가 출력 해줘
```

### namespace 사용
```cpp
using namespace std; // std라는 문자를 정의해준것
```

원래는 다음과 같이 사용해야 함:
```cpp
std::cout << "안녕하세요!" << std::endl;
```

`iostream` 라이브러리가 그렇게 하라고 되어있음

## 문자형 (char) 타입

### 기본 사용법
- `char` 타입은 작은 따옴표 `''`를 사용
- 큰 따옴표 `""`는 사용할 수 없음

```cpp
char a = 'a';   // 가능
char a = "a";   // 불가능
```

### char 배열과 NULL 문자
```cpp
char a[5] = { 'a', 'b', 'c' };  // 에러 발생
char a[] = { 'a', 'b', 'c', '\0' };  // 정상 - NULL값 필요
```

## 문자열 처리

### char vs string 비교
```cpp
char a[20];
char b[20] = "jauar";
string a1;
string b1 = "panda";

// a = b;       // char형은 문자열 값을 못줌
a1 = b1;        // string 타입은 문자열 값을 줄 수 있음

cout << a1 << endl;       // panda 출력
cout << a1[0] << endl;    // p 출력
```

### 공백이 포함된 문자열 입력
```cpp
char a[20];
cin >> a;   // 입력: Hello world!
// 출력: Hello (공백 이후 무시됨)

// 해결방법
cin.get(a, 20);        // 공백 포함하여 입력
cin.getline(a, 20);    // 공백 포함하여 입력

// 특정 문자까지만 입력받기
cin.getline(a, 20, ';');   // 세미콜론까지만 입력받음
// 입력: Hello; world! → 출력: Hello
```

### string 타입 입력
```cpp
string a;
getline(cin, a);  // 공백 포함하여 입력받기
```

## string 타입 상세

### 선언 방법
```cpp
string a = "Hello";                    // 정상
string a[5] = "Hello";                 // 에러
string a[3] = { "Hello", "C++", "world" };  // 배열 선언
```

### 포인터 배열과의 관계
```cpp
string a[3] = { "Hello", "C++", "world" };
// 위 코드는 아래와 동일
char* a[3] = { "Hello", "C++", "world" };
```

### 동적 할당
```cpp
string* a = new string;        // 단일 객체
string* a = new string[3];    // 배열
```

### 생성자 사용
```cpp
string a("안녕하세요");  // 생성자를 통한 초기화
```

## 주요 포인트
1. **char vs string**: char는 작은 따옴표, string은 큰 따옴표
2. **NULL 문자**: char 배열은 `\0` 필요
3. **공백 처리**: `cin.get()` 또는 `getline()` 사용
4. **동적 할당**: `new` 연산자로 메모리 할당 가능
5. **생성자**: string 타입은 생성자로 초기화 가능
