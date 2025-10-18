# 모던 C++ 람다 함수

## 람다 함수

람다 함수는 익명 함수(이름이 없는 함수)로 짧고 간결한 코드를 작성할 때 유용해 특히 콜백 함수, 알고리즘(STL), 멀티스레딩 같은 곳에서 많이 쓰여

### 람다 함수 기본 문법
```cpp
[캡처](매개변수) -> 반환 타입 { 함수 내용 }
```

- **[캡처]**: 함수 외부의 변수를 사용할지 결정
- **(매개변수)**: 일반 함수처럼 매개변수 사용 가능
- **-> 반환 타입**: 생략 가능(컴파일러가 추론)
- **{ 함수 내용 }**: 실행할 코드 불록

### 예시 코드
```cpp
#include <iostream>

using namespace std;

int main() {
    auto add = [](int a, int b) { return a + b; };
    cout << add(3, 5) << endl;   // 출력 값: 8
}
```

`add`가 람다 함수로 `[](int a, int b) { return a + b; };` 이거를 저장하고 있음

### 람다 함수를 출력할 때는 몇가지 방법이 있는데

```cpp
int main() {
    int x = 10;

    auto lambda = [x]() { return x * x; };

    lambda(); 
}
```

이런식으로 그냥 파이썬처럼 lambda만 호출해서 출력할 수 있음 대신 위에 있는건 오류는 안걸리는데 아무것도 출력이 되지는 않음

```cpp
int main() {
    int x = 10;

    auto lambda = [x]() { cout << x << endl; };

    lambda(); 
}
```

이런식으로 출력을 할 수도 있음

```cpp
int main() {
    int x = 10;
    auto lambda = [x]() { return x * 2; }; // 값을 반환하는 람다

    cout << lambda() << endl; //  정상 출력 (20)
    //cout << lambda << endl;   // 오류!
}
```

이런식으로 출력을 할 수 있다 근데 `cout << lambda << endl;` 이런식으로 출력을 하면 에러가 걸림 람다 함수는 객체(클래스)처럼 동작을 하는데 ostream에는 람다 함수를 출력하는 연산자가 정의 된게 없어서 오류가 걸림 그래서 출력을 할려면 꼭 ()괄호를 붙여야 함

그리고
```cpp
int main() {
    int x = 10;

    auto lambda = [x]() { cout << "x = " << x << endl; }; 

    cout << lambda() << endl; // ❌ 컴파일 오류 발생!
}
```

이런 경우에도 오류가 걸림 왜냐면 람다 함수에서 반환 값이 없어서 오류가 걸림
그래서 
```cpp
int main() {
    int x = 10;

    auto lambda = [x]() { 
        cout << "x = " << x << endl;  //  출력 발생
        return x; //  반환값 있음
    }; 

    cout << lambda() << endl; //  정상 동작
}
```

이런식으로 해야 오류가 안걸리고 출력이 됨

## 캡처 방식

캡처 방식은 기본적으로 4가지가 있다

1. 값 캡처 ([=] 또는 [변수명])
2. 참조 캡처 ([&] 또는 [&변수명])
3. 혼합 캡처 ([=, &변수명] 또는 [&, 변수명])
4. 이동 캡처 ([변수명 = move(변수)])

### 1. 값 캡처 [=] 또는 [변수명]

- 람다 함수가 바깥 변수의 복사본을 저장해서 사용해.
- 람다 내부에서 값을 변경해도 원본은 변하지 않아!
- 값 캡처는 람다 함수 안에서 값을 변경 못함.

```cpp
int main() {
    int x = 10;

    auto lambda = [x]() { cout << "x = " << x << endl; };

    x = 20; // 원본 변수 변경
    lambda(); // 여전히 x = 10 (복사본 사용)  
}
```

`[x]` 덕분에 x의 복사본이 람다 내부에서 사용돼.
원본 x를 20으로 바꿔도, 람다는 여전히 x = 10을 출력해!
값 캡처기 떄문에 람다 함수 안에서 값을 못 바꿈 `x = 20;` 이렇게 하면 컴파일 오류가 남

근데 만약 값 캡처인데 람다 함수 안에서 값을 바꾸고 싶다면 mutable을 사용하면 값 캡처여도 변수를 변경할 수 있다.

```cpp
auto lambda = [x]() mutable {
    x = 20;  // x 값을 수정 가능
    cout << "Modified x: " << x << endl;
};
```

대신에 람다 함수 밖으로 나오면 값은 외부에 값 그대로이다

```cpp
int x = 10;

auto by_value = [x]() mutable {  // mutable 키워드를 사용하면 값 캡처된 변수 수정 가능
    x = 20;  // x 값을 수정
    cout << "Inside by_value: " << x << endl;    // 20
};
by_value();
cout << "Outside by_value: " << x << endl;  // x는 여전히 10
```

### 2. 참조 캡처 [&] 또는 [&변수명]

- 람다 함수가 바깥 변수의 원본을 직접 사용해.
- 람다 내부에서 값을 변경하면 원본도 바뀌어!

```cpp
int main() {
    int x = 10;

    auto lambda = [&x]() { x *= 2; }; // x를 2배로 변경

    lambda();
    cout << "x = " << x << endl; // x = 20
}
```

`[&x]` 덕분에 람다가 원본 x를 직접 수정 가능해!
lambda() 실행 후, x가 20으로 바뀌었어.

### 3. 혼합 캡처 [=, &변수명] 또는 [&, 변수명]

- `[=, &y]` → 기본적으로 값 캡처하지만, y는 참조 캡처
참조하는 y를 제외하고 다 값 캡처라서 y만 람다 함수 안에서 값을 변경을 할 수 있고 나머지는 값 캡처기 때문에 값을 변경하면 오류가 걸림

- `[&, x]` → 기본적으로 참조 캡처하지만, x는 값 캡처
나머지는 다 참조로 값을 받고 x만 값 캡처만 받음 그니까 x를 제외한 나머지는 람다 함수 안에서 값을 변경할 수 있지만 x는 값 캡처기 때문에 값을 변경 못함

```cpp
int main() {
    int a = 10, b = 20;

    auto lambda = [=, &b]() { 
        // a는 값 캡처 (변경 불가)
        // b는 참조 캡처 (변경 가능)
        // a *= 2; // 오류!
        b *= 2;
    };

    lambda();
    cout << "a = " << a << ", b = " << b << endl; // a = 10, b = 40
}
```

a는 복사본 사용(변경 불가), b는 참조로 사용(변경 가능)

### 4. 이동 캡처 [변수명 = move(변수)]

- 객체를 이동해서 캡처할 때 사용
- std::unique_ptr 같은 이동 전용 객체 캡처 가능

```cpp
#include <iostream>
#include <memory>

using namespace std;

int main() {
    auto ptr = make_unique<int>(42);

    auto lambda = [ptr = move(ptr)]() { // ptr을 이동 캡처
        cout << "값: " << *ptr << endl;
    };

    lambda();
    // cout << *ptr; // 오류! ptr은 이동됨
}
```

move(ptr) 덕분에, ptr이 람다로 이동되고 원본은 사용 불가
여기서 `ptr = move(ptr)` 여기 부분이 이해가 안갈 수 있다 쉽게 말하면 main에 있는 ptr에 소유권이 람다 함수 안에 있는 ptr에 변수에 소유권이 넘어간 상태이다 그래서 람다 함수 밖에서 사용하면 오류가 걸리는 이유가 소유권이 람다 함수 안에있는 ptr한테 있기 때문에 밖에 있는 ptr이 출력을 하면 오류가 걸림 그러면 `return ptr;` 이런식으로 반환 하면 밖에서 사용할 수 있지 않냐? 라고 물어볼 수 있지만 그래도 소유권이 이미 람다 안에 ptr한테 넘어간 상태여서 오류가 걸림 대신에 이런식으로

```cpp
auto ptr = make_unique<int>(42);

auto lambda = [ptr = move(ptr)]() {
    cout << "값: " << *ptr << endl;
    return ptr;  // ptr을 반환
};

auto returnedPtr = lambda();  // ptr을 반환받음
cout << *returnedPtr << endl;  // 람다 내에서 ptr의 값은 여전히 출력 가능

// cout << *ptr << endl;  // 오류! ptr은 이미 이동됨
```

이런식으로 반환을 사용하면 사용을 할 수 있긴 하지만 그래도 밖에 있는 ptr은 사용 불가

람다 함수랑 마찬가지로 일반 함수도 똑같이 적용

```cpp
void processUniquePtr(std::unique_ptr<int> ptr) {
    cout << "ptr의 값: " << *ptr << endl;
    // ptr은 이 함수 내에서만 유효
}

int main() {
    auto ptr = make_unique<int>(42);

    processUniquePtr(move(ptr));  // ptr을 이동

    // cout << *ptr << endl;  // 오류! ptr은 이미 이동되어 유효하지 않음
}
```

## 캡처 + 매개변수 + 반환 타입

```cpp
int main() {
    int x = 10;

    // 반환 타입을 -> int로 명시
    auto lambda = [x](int a, int b) -> int {    // (-> int) 반환 타입
        return x + a + b;
    };

    cout << lambda(3, 5) << endl; // x(10) + 3 + 5 = 18
}
```

**출력 값:** 18

### 반환 타입을 생략해도 되는 경우

- 만약 람다 내부에서 반환값이 명확하면 반환 타입 생략 가능!

```cpp
auto lambda = [x](int a, int b) { return x + a + b; };   // OK!
```

이렇게 해도 자동으로 int 반환으로 추론됨.

```cpp
int main() {
    auto lambda = [](bool flag) -> double {
        if (flag)
            return 3.14;  // double
        else
            return 2;      // int  오류가 걸림
    };

    cout << lambda(true) << endl;  // 3.14
    cout << lambda(false) << endl; // 2 
}
```

여기서 반환 타입을 double로 했는데 `return 2`에서 2는 int형이기 때문에 오류가 걸려 그래서

```cpp
else
    return static_cast<double>(2); // int -> double로 변환
```

이런식으로 타입을 하나로 맞춰주거나 아니면

```cpp
auto lambda = [](bool flag) -> auto 이걸로 쓰면 타입이 자동으로 맞춰져서 반환 할 수 있어
```

## 문제 해결

### 문제1
```cpp
#include <iostream>

using namespace std;

int main() {
    int x = 10;
    int y = 20;

    auto by_value = [x, y]() {
        cout << "by_value - x: " << x << ", y: " << y << endl;
    };

    auto by_reference = [&x, &y]() {
        x += 5;
        y += 10;
        cout << "by_reference - x: " << x << ", y: " << y << endl;
    };

    auto mixed_capture = [x, &y]() {
        //x += 5;  // x는 캡처된 값이므로 수정 불가
        y += 10; // y는 참조로 캡처되어 수정 가능
        cout << "mixed_capture - x: " << x << ", y: " << y << endl;
    };

    by_value();
    by_reference();
    mixed_capture();

    cout << "Outside - x: " << x << ", y: " << y << endl;
    
    return 0;
}
```

**출력 값:**
```
by_value - x: 10, y: 20
by_reference - x: 15, y: 30
mixed_capture - x: 10, y: 40
Outside - x: 15, y: 40
```

여기서 어라? 왜 mixed_capture x 값이 10이야?? 15가 되어야 하는거 아니야? 라고 할 수 있다 왜냐면 전에 레퍼런스 값이 15였는데 mixed_capture 람다 함수로 와서도 x 값이 15어야 되는거 아니야? 라고 생각 할 수 있지만 람다 함수에서 값을 받을 때 참조 되는게 아니면 기본 변수 초기 값으로 받는다.
즉 mixed_capture y 값이 30 그대로 온 이유는 mixed_capture에서 값을 받을 때 y를 참조해서 받았기 때문에 값이 30 그대로 옮겨져서 40이 될 수 있었음 근데 x는 값 캡처로 받았어서 변수 초기 값으로 받음 그래서 10으로 값을 받아서 출력도 10이 됨

### 문제2
```cpp
#include <iostream>
#include <functional>

using namespace std;

int main() {
    // 람다를 std::function에 저장하고 호출하는 코드 작성
    return 0;
}
```

```cpp
#include <iostream>
#include <functional>

using namespace std;

int main() {
    // 람다를 std::function에 저장하고 호출하는 코드 작성
    int a = 3;
    std::functional<int> func1 = [a]() {
        if (a == 0) {
            return a;
        }
        func1(a - 1);
        cout << a << endl;
    };
 
    return 0;
}
```

근데 이러면 오류가 걸림 왜냐면 자기 자신을 호출하는 함수를 만들어야 하는데 지금 캡처를 보면 값 캡처라서 변경이 불가능 하고 또 func1안에 있는 변수로 해야 함 그래서

```cpp
#include <iostream>
#include <functional>
using namespace std;

int main() {
    int a = 3;
    // std::function을 사용해 람다 저장
    std::function<void(int)> func1 = [&](int n) {  // 반환타입을 void로 지정
        if (n == 0) {
            return;
        }
        cout << n << endl;
        func1(n - 1);  // 재귀 호출
    };
    
    func1(a);  // func1을 호출하고 출력이 시작됨
    return 0;
}
```

## 람다와 STL 알고리즘

람다는 특히 순회 및 처리에서 매우 유용해. 예를 들어, std::for_each를 사용할 때 람다를 넘겨줄 수 있어.

```cpp
#include <iostream>
#include <vector>
#include <algorithm>  // std::for_each 헤더 포함
using namespace std;

int main() {
    vector<int> nums = {1, 2, 3, 4, 5};

    // 람다를 사용하여 각 원소 출력
    for_each(nums.begin(), nums.end(), [](int num) {
        cout << num << " ";
    });
    cout << endl;

    return 0;
}
```

for_each 이거는 반복문을 더 함수형 스타일로 사용할 수 있도록 도와주는 함수
- 람다, 함수 포인터에서도 활용 가능
- const 요소 사용
- 컴파일러 최적화 가능
- STL 컨테이너 활용

```cpp
for_each(numbers.begin(), numbers.end(), [](int num) {
    cout << num << " ";
});
```

**장점:** 간결하고 직관적
**단점:** 요소를 수정하려면 auto&로 받거나 transform을 사용해야 함

```cpp
for_each(numbers.begin(), numbers.end(), [](int &num) { num *= 2; });
```

num을 참조&로 받으면 값을 변경 가능

### 함수 포인터 사용
```cpp
void print(int num) {
    cout << num << " ";
}

for_each(numbers.begin(), numbers.end(), print);
```

람다 대신 일반 함수로 사용 가능

## 람다 함수의 this 포인터

람다 함수 내부에서 객체의 멤버 함수에 접근하려면 this 포인터를 사용해야 할 때도 있어. 이때, this 포인터는 람다 함수가 객체에 대한 참조를 캡처하는 방식으로 접근할 수 있어.

### 예시:
```cpp
#include <iostream>
using namespace std;

class MyClass {
public:
    void show() {
        auto lambda = [this]() { cout << "Value: " << value << endl; };  // 
        lambda();
    }

    int value = 10;
};

int main() {
    MyClass obj;
    obj.show();  // 10 출력
    return 0;
}
```

**출력 값:** Value: 10

- `[this]` → 현재 객체의 this 포인터를 캡처
- `cout << "Value: " << value << endl;` → `this->value`와 동일 (즉, obj.value에 접근)
- 결과적으로 obj.value == 10이므로 "Value: 10"이 출력됨

## std::thread와 람다

람다는 멀티스레딩에서 유용하게 사용할 수 있어. 예를 들어, std::thread와 함께 사용하면 스레드 내에서 실행될 함수를 간결하게 정의할 수 있어.

### 예시:
```cpp
#include <iostream>
#include <thread>
using namespace std;

int main() {
    thread t([]() { 
        cout << "Hello from the thread!" << endl; 
    });
    
    t.join();  // 메인 스레드가 종료되기 전에 t 스레드가 종료되도록 기다림

    return 0;
}
```

## 재귀 함수 X

람다 함수는 재귀 함수가 불가능
람다는 기본적으로 자신을 참조하는 방법이 없기 때문에
만약 재귀 함수를 사용하려면 std::function 이걸을 사용해야 해

```cpp
auto factorial = [](int n) {
        if (n == 0)
            return 1;
        return n * factorial(n - 1);  // 컴파일 오류
    };

    std::cout << factorial(5) << std::endl;
```

이런식으로 재귀를 사용하면 오류가 걸려서 람다를 재귀를 사용할 때는

```cpp
#include <iostream>
#include <functional>

int main() {
    // std::function을 이용한 재귀 가능
    std::function<int(int)> factorial = [&](int n) {
        if (n == 0)
            return 1;
        return n * factorial(n - 1);  // 재귀 호출
    };

    std::cout << factorial(5) << std::endl;  // 120
    return 0;
}
```

이런식으로 사용해야 함
std::function은 함수 포인터처럼 자기 자신을 참조할 수 있도록 할 수 있습니다.

std::function은 일반 함수 포인터와 달리 람다 함수, 함수 객체 등을 캡슐화할 수 있는 유연한 타입입니다. 그래서 람다 함수가 자기 자신을 참조할 수 있게 만들어주는 것이죠.

std::function을 사용하면 람다 함수에서 재귀 호출을 할 수 있게 됩니다.

람다 함수 자체에서는 재귀를 직접 사용할 수 없지만, std::function을 사용하면 람다 함수 안에서 재귀를 구현할 수 있어요!

### std::function 기본 사용법
```cpp
#include <iostream>
#include <functional>  // std::function 헤더 포함

void greet() {
    std::cout << "Hello, world!" << std::endl;
}

int add(int a, int b) {
    return a + b;
}

int main() {
    // 함수 포인터를 std::function으로 저장
    std::function<void()> f1 = greet;
    f1();  // Hello, world!

    // 람다 함수 저장
    std::function<int(int, int)> f2 = [](int a, int b) { return a + b; };
    std::cout << f2(3, 4) << std::endl;  // 7

    return 0;
}
```

이건 std::function 대한 설명이야
void가 반환 값이야

### 함수 객체 저장
함수 객체는 operator()를 오버로드하여 함수처럼 호출 가능한 객체를 만드는 방법입니다. std::function은 함수 객체도 저장할 수 있습니다.

```cpp
#include <iostream>
#include <functional>

class Multiply {
public:
    int operator()(int a, int b) {
        return a * b;
    }
};

int main() {
    std::function<int(int, int)> multiplyFunc = Multiply();
    std::cout << multiplyFunc(3, 4) << std::endl;  // 12

    return 0;
}
```

### 멤버 함수와 std::function:

멤버 함수 포인터도 std::function에 저장하고 호출할 수 있습니다. 멤버 함수는 클래스의 인스턴스를 통해 호출되어야 하므로, std::function에 멤버 함수 포인터를 바인딩할 때는 객체도 함께 바인딩해야 합니다.

```cpp
#include <iostream>
#include <functional>

class Calculator {
public:
    int add(int a, int b) {
        return a + b;
    }
};

int main() {
    Calculator calc;

    // 멤버 함수 포인터와 객체를 바인딩
    std::function<int(Calculator&, int, int)> addFunc = &Calculator::add;

    // 멤버 함수 호출
    std::cout << addFunc(calc, 3, 4) << std::endl;  // 7

    return 0;
}
```

여기서 `&Calculator::add;` 만으로는 어떤 객체에서 호출할지 모른다는 점에서 `Calculator&` 이걸로 객체참조를 명시적으로 전달해야해 멤버 함수는 그래야 제대로 동작할 수 있어

만약에 `int (Calculator::*funcPtr)(int, int) = &Calculator::add;` 이런식으로 하면 바로 호출을 할 수가 없어 이건 그냥 멤버 함수 포인터일 뿐 반드시 어떤 객체의 멤버 함수인지 지정해야 함

### std::function의 타입 추론:

std::function을 사용할 때 타입을 명시적으로 지정할 수도 있지만, 람다 함수는 auto를 사용하여 타입을 자동으로 추론할 수 있습니다.

```cpp
#include <iostream>
#include <functional>

int main() {
    auto func = std::function<int(int, int)>([](int a, int b) { return a + b; });
    std::cout << func(5, 3) << std::endl;  // 8

    return 0;
}
```
