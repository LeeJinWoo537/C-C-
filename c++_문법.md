# C++ 문법 심화

## 인라인 함수 (Inline Function)

### 개념
일반 함수를 호출하는 것과 출력하는 건 다르지 않지만, 인라인 함수를 쓰면 오버헤드(추가로 드는 시간이나 자원)를 감소시킵니다.

인라인 함수는 함수 호출을 아예 없애고, 호출된 위치에 함수의 코드를 복사해서 넣습니다. 그래서 함수 호출 시 필요한 작업들이 없으므로 오버헤드가 줄어듭니다.

**단점**: 코드가 커질 수 있어서 더 많은 메모리 공간이 필요할 수 있음

### 사용법
```cpp
inline int Function(int a) {
    // 함수 내용
}
```

## 디폴트 매개변수 (Default Parameters)

### 기본 개념
함수 매개변수에 기본값을 미리 설정하는 기능

### 예제
```cpp
void Function(std::string name = "Hello", int age = 20) {
    std::cout << "Hello, " << name << "! You are " << age << " years old." << std::endl;
}

int main() {
    Function();                     // 매개변수를 넘기지 않음 -> 기본값 사용
    Function("Alice");              // 이름만 넘김 -> 나이는 기본값 사용
    Function("Bob", 25);            // 이름과 나이 모두 넘김 -> 넘긴 값 사용

    return 0;
}
```

**출력 결과:**
```
Hello, Hello! You are 20 years old.
Hello, Alice! You are 20 years old.
Hello, Bob! You are 25 years old.
```

### 디폴트 매개변수 규칙
- 오른쪽 매개변수부터 기본값 설정
- 왼쪽 매개변수는 main에서 받을 변수

```cpp
void Function(int a, int b = 1) {  // 오른쪽에 기본값 설정
    return a + b;
}

int main() {
    int a = 1;
    Function(a);  // b는 기본값 1 사용
    return 0;
}
```

## 참조 변수 (Reference Variables)

### 기본 개념
참조 변수는 포인터와 비슷하게 변수를 가리킬 때 값이 같이 변한다는 점에서 동일하지만, 사용법이 다릅니다.

### 예제
```cpp
int a = 10;         // 원래 변수
int& b = a;         // 'a'에 대한 참조 변수 'b'

cout << "a: " << a << endl;       // 출력: 10 
cout << "b: " << b << endl;      // 출력: 10

b = 20;              // 'b'를 통해 'a'의 값을 변경
cout << "a or b: " << a << endl;   // 출력: 20
```

### 참조 변수 vs 포인터 비교

| 특징 | 참조 변수 | 포인터 |
|------|-----------|--------|
| 초기화 | 필수적 (선언시 반드시 지정) | 선택적 (나중에 초기화 가능) |
| 주소 연산자 | 사용 안함 | 사용함 (&, * 연산자) |
| 참조 변경 | 불가능 | 가능 |
| 사용법 | 간결하고 직관적 | 복잡함 |
| NULL 값 | 가질 수 없음 | 가질 수 있음 |

### 참조 변수 사용 이유
1. **매개변수로 전달**: 큰 데이터를 복사하지 않고 원본 데이터를 참조
2. **리턴값으로 사용**: 함수를 통해 참조값을 반환하여 원본 데이터를 변경
3. **간결한 방식**: 함수 인자로 전달할 때 간결한 방식

## 함수 오버로딩 (Function Overloading)

### 개념
여러 개의 함수를 같은 이름으로 연결한다. 함수를 호출할 때는 매개변수(함수에 있는 변수)를 보고 판단한다.

### 예제
```cpp
void Function(char, int);  // 매개변수를 보고 판단해서 값을 호출
void Function(int, int);
void Function(char);

int main() {
    Function('a', 3);
    Function(3, 2);
    Function('a');
    
    return 0;
}
```

### 타입별 오버로딩
```cpp
int Function(int, int);
float Function(float, float);

int main() {
    int a, b;
    float c, d;

    cin >> a >> b;     // 입력: 1, 2
    cin >> c >> d;     // 입력: 1.1, 1.2

    cout << Function(a, b) << endl;   // 출력: 3
    cout << Function(c, d) << endl;   // 출력: 2.3
    
    return 0;
}

int Function(int a, int b) {
    return a + b;
}

float Function(float c, float d) {
    return c + d;
}
```

### 함수 오버로딩 제한사항

#### 1. 리턴형만 다른 경우는 불가능
```cpp
int Function(int a, int b) {
    return a + b;
}

double Function(int a, int b) {  // 오류 발생 - 매개변수가 동일
    return a + b;
}
```

**정상적인 경우:**
```cpp
int add(int a, int b) {
    return a + b;
}

double add(double a, double b) {  // 정상 - 매개변수 타입이 다름
    return a + b;
}
```

#### 2. 모호한 호출 방지
```cpp
cout << Function(1, 1.1) << endl;  // 정수와 실수 혼합 - 모호함
```

명확하게 타입을 지정해야 함:
```cpp
int a = 1, b = 2;
float c = 1.1, d = 1.2;

cout << Function(a, b) << endl;   // 명확
cout << Function(c, d) << endl;   // 명확
```

## 함수 템플릿 (Function Template)

### 개념
일반화 프로그래밍 - 구체적인 데이터형을 포괄할 수 있는 일반형으로 함수를 정의

### 기본 사용법
```cpp
template <class a>  // class 대신 타입이름을 써도 괜찮음
a Function(a, a);

int main() {
    int num1 = 1, num2 = 2;
    cout << Function(num1, num2) << endl;    // 정수형으로 호출

    float num3 = 1.1, num4 = 1.2;
    cout << Function(num3, num4) << endl;    // 실수형으로 호출
    
    return 0;
}

template <class a>
a Function(a num1, a num2) {
    return num1 + num2;
}
```

**출력 결과:**
```
3
2.3
```

### 서로 다른 타입 사용
```cpp
template <class a>
a Function(int, a);   // 하나는 다른 타입을 두고 사용

int main() {
    int num1 = 1;
    float num3 = 1.1;
    
    cout << Function(num1, num3) << endl;   // 출력: 2.1
    
    return 0;
}

template <class a>
a Function(int num1, a num2) {
    return num1 + num2;
}
```

### 템플릿 함수 오버로딩
```cpp
template <class a>
a Function(a, a);

template <class a>
a Function(int, a);

int main() {
    int num1 = 1, num2 = 2;
    // cout << Function(num1, num2) << endl;  // 오류 - 같은 타입의 함수가 2개

    float num3 = 1.1, num4 = 1.2;
    cout << Function(num3, num4) << endl;   // 출력: 2.3

    cout << Function(num1, num3) << endl;   // 출력: 2.1
    
    return 0;
}

template <class a>
a Function(a num1, a num2) {
    return num1 + num2;
}

template <class a>
a Function(int num1, a num2) {
    return num1 + num2;
}
```

## 주요 포인트
1. **인라인 함수**: 오버헤드 감소, 코드 크기 증가
2. **디폴트 매개변수**: 오른쪽부터 기본값 설정
3. **참조 변수**: 포인터보다 간결하고 안전
4. **함수 오버로딩**: 매개변수 타입으로 구분
5. **함수 템플릿**: 타입에 독립적인 일반화 프로그래밍
