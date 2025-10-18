# C++ 클래스 (Class)

## 개요
C++ 클래스는 객체지향 프로그래밍의 핵심 개념으로, 데이터와 함수를 하나의 단위로 묶어서 관리합니다.

## 클래스 기본 구조

### 접근 제어 지시자
클래스 안에 접근 제어 지시자(private, public) 없이 변수나 함수를 만들 수 없음

#### private
1. 클래스 안에 있는 것으로 변수나 메소드(함수)를 생성할 수 있음
2. 클래스 외부에서 private 변수나 함수에 직접 접근할 수 없고, 클래스 내부의 public 함수나 클래스 안에 있는 함수를 통해서만 간접적으로 접근할 수 있음
3. 주로 내부 데이터를 보호하기 위해 사용됨
4. private끼리도 접근이 가능함

#### public
1. 클래스 안에 있는 것으로 변수나 메소드(함수)를 생성할 수 있음
2. 외부에서 자유롭게 접근할 수 있음
3. 클래스 외부의 함수나 코드에서도 public 안에 있는 변수나 함수를 직접 접근하거나 호출할 수 있음
4. 주로 외부에 제공할 기능을 정의하거나 외부에서 필요한 데이터에 접근할 수 있도록 공개된 인터페이스 역할

### 클래스 정의 예제
```cpp
class Stock {
private:
    string name;
    int shares;
    float share_val;
    double total_val;
    void set_total() {
        total_val = shares * share_val; 
    }

public:
    void acquire(string, int, float);      // "Panda", 100, 1000
    void buy(int, float);
    void sell(int, float);
    void update(float);
    void show();
    Stock();
    ~Stock();
};
```

### 멤버 함수 정의
`::` 연산자는 접근한다는 뜻으로, 구조체에 있는 `.`(점)과 똑같은 것이다.

```cpp
void Stock::acquire(string co, int n, float pr) {    // "Panda", 100, 1000
    name = co;        // .(점)이나 :: 이런게 없어도 클래스 내부에 있는 private함수나 이런것도 자유롭게 사용이 가능하다
    shares = n;
    share_val = pr;
    set_total();
}

void Stock::buy(int n, float pr) {
    shares += n;
    share_val = pr;
    set_total();
}

void Stock::sell(int n, float pr) {
    shares -= n;
    share_val = pr;
    set_total();
}

void Stock::update(float pr) {
    share_val = pr;
    set_total();
}

void Stock::show() {
    cout << "회사 명: " << name << endl;
    cout << "주식 수: " << shares << endl;
    cout << "주가: " << share_val << endl;
    cout << "주식 총 가치: " << total_val << endl;
}
```

### 사용 예제
```cpp
int main() {
    Stock temp;
    temp.acquire("Panda", 100, 1000);
    temp.show();
    temp.buy(10, 1200);
    temp.show();
    temp.sell(5, 800);
    temp.show();
    
    return 0;
}
```

## 생성자 (Constructor)

### 개념
1. 생성자는 클래스가 객체로 생성될 때 자동으로 호출되는 특별한 함수
2. 객체가 생성될 때 멤버 변수(클래스 안에 생성된 변수)를 초기화하거나 객체 생성시 필요한 작업을 수행하는 데 사용
3. 생성자의 이름은 클래스의 이름과 동일하다
4. 생성자는 반환형(return)이 없다, 리턴 값이 없다
5. 생성자는 매개변수를 가질 수도 있고 가질 수 없기도 하다
6. 매개변수가 없는 생성자를 디폴트 생성자라고 한다
7. 생성자를 명시적으로 정의하지 않으면, 컴파일러가 자동으로 디폴트 생성자를 제공한다

### 생성자 정의
```cpp
Stock::Stock() {       // 생성자
    
}

Stock::~Stock() {    // 파괴자
    
}
```

### 생성자 호출 방법
```cpp
Stock temp = Stock("Panda", 100, 1000);   // 방법1
Stock temp2("Panda", 100, 1000);            // 방법2
```

### 디폴트 생성자 예제
```cpp
class Stock {
private:
    string name;
    int shares;

public:
    // 생성자 (클래스 이름과 동일)
    Stock() {
        name = "Hello";       // 생성자는 이러한 멤버 변수를 초기화하는 역할을 한다
        shares = 0;
    }
};

int main() {
    Stock stock1;         // Stock() 생성자가 호출됨
    return 0;
}
```

## 소멸자 (Destructor)

### 개념
1. 소멸자는 객체가 더 이상 필요하지 않아 메모리에서 해제될 때 자동으로 호출되는 함수
2. 객체가 소멸될 때 객체가 차지하고 있던 리소스를 반환하거나 정리할 작업을 수행
3. 소멸자의 이름은 클래스 이름 앞에 `~` 기호를 붙여서 정의한다
4. 소멸자는 반환형(return)이 없고 매개변수를 가질 수 없다

### 소멸자 예제
```cpp
class Stock {
private:
    string name;
    int shares;

public:
    Stock() {
        name = "Hello";
        shares = 0;
    }

    // 소멸자 (클래스 이름 앞에 ~ 기호가 붙음)
    ~Stock() {
        cout << "소멸자가 호출됩니다!" << endl;
    }
};

int main() {
    Stock stock1;  // 객체가 생성됨 -> 생성자가 호출됨
}  // 객체가 범위를 벗어나면 -> 소멸자가 호출됨
```

## this 포인터

### 개념
C++에서 객체 자신을 가리키는 포인터입니다.

클래스의 멤버 함수 내부에서, `this`는 현재 호출된 객체의 주소를 가리킵니다. 이를 통해 멤버 함수 내에서 객체 자신의 데이터나 함수를 조작하거나 참조할 수 있습니다.

### 특징
1. **현재 객체의 주소를 가리킴**: `this`는 클래스 내의 모든 비정적 멤버 함수에 자동으로 전달되며, 현재 객체의 주소를 담고 있습니다.
2. **멤버 변수와 매개변수 구분**: 멤버 함수에서 매개변수 이름이 멤버 변수 이름과 동일할 때, `this`를 사용해 멤버 변수를 명시적으로 참조할 수 있습니다.
3. **체인 호출(메소드 체이닝) 가능**: 멤버 함수가 `*this`를 반환하면, 메소드 체이닝을 구현할 수 있습니다.

### 기본 사용법
```cpp
class MyClass {
private:
    int value;      // 멤버 변수

public:
    void setValue(int value) {    // 매개변수
        this->value = value;      // this->value는 멤버 변수, value는 매개변수
    }
};
```

### 체이닝 예제
```cpp
class a {
private:
    int value;

public:
    // 생성자
    a(int value) {
        // this를 사용하여 멤버 변수와 매개변수 구분
        this->value = value;
    }

    // 멤버 함수
    void setValue(int value) {
        this->value = value;  // this를 사용하여 멤버 변수에 접근
    }

    // 멤버 함수 체이닝 예시
    a& increment() {
        this->value += 1;
        return *this;  // this 포인터를 역참조하여 객체 자체를 반환
    }

    void show() {
        std::cout << "Value: " << this->value << std::endl;
    }
};

int main() {
    a obj(5);
    obj.increment().increment().show();  // 메소드 체이닝
    return 0;
}
```

### 참조 반환과 역참조
- `a&`: 객체에 대한 참조를 반환한다는 뜻
- `return *this`: `this` 포인터를 역참조하여 객체 자체를 반환

**참조를 반환하는 이유**: 참조를 반환하면 함수 호출 후에 객체 자체를 다시 사용할 수 있게 된다.

### 주소 출력 예제
```cpp
class MyClass {
public:
    void PrintThis() {
        cout << "나의 주소는 " << this << endl;
    }
};

int main() {
    MyClass a, b;

    cout << "a의 주소는 " << &a << endl;
    cout << "b의 주소는 " << &b << endl;

    a.PrintThis();
    b.PrintThis();
    
    return 0;
}
```

## 클래스 배열

### 기본 사용법
```cpp
class Stock {
private:
    string name;
    int shares;
    float share_val;
    double total_val;
    void set_total() {
        total_val = shares * share_val; 
    }

public:
    void acquire(string, int, float);
    void buy(int, float);
    void sell(int, float);
    void update(float);
    void show();
    Stock topval(Stock&);
    Stock(string, int, float);
    Stock();
    ~Stock();
};

// 배열 사용 예제
int main() {
    Stock s[4] = {
        Stock("A", 10, 1000),
        Stock("B", 20, 1200),
        Stock("C", 30, 1300),
        Stock("D", 40, 1400)
    };

    Stock first = s[0];

    for (int i = 1; i < 4; i++) {
        first = first.topval(s[i]);   // topval에 리턴 값을 저장하게된다면 0번째랑 1번째 비교 그다음 2번째랑 비교 그다음 3번째 비교
    }
    first.show();
    
    return 0;
}
```

### 참조 반환을 사용한 클래스 배열
```cpp
class Stock {
public:
    Stock &topval(Stock&);   // 참조 값 반환
    // ... 다른 멤버들
};

Stock &Stock::topval(Stock& s) {     // 클래스에도 참조 값을 해줌 그래서 객체 자체가 반환
    if (s.share_val > share_val) {
        return s;
    } else {
        return *this;
    }
}

int main() {
    Stock s[4] = {
        Stock("A", 10, 1000),
        Stock("B", 20, 1200),
        Stock("C", 30, 1300),
        Stock("D", 40, 1400)
    };

    Stock* first = &s[0];

    for (int i = 1; i < 4; i++) {
        first = &first->topval(s[i]);   // 포인터를 사용한 접근
    }
    first->show();

    return 0;
}
```

### 클래스 배열의 중요 사항
- **디폴트 생성자 필요**: 배열 원소들에게 각각 또 다른 생성자를 할당해주기 때문에 클래스를 배열로 선언하려면 꼭 디폴트 생성자가 있어야 한다.

```cpp
Stock s[4] = {
    Stock("A", 10, 1000),
    Stock("B", 20, 1200),
    Stock("C", 30, 1300),
    Stock("D", 40, 1400)
};

s[0].show();    // 인덱스 0번째 배열이 출력 된다.
```

## 주요 포인트
1. **접근 제어**: private(내부), public(외부 접근 가능)
2. **생성자/소멸자**: 객체 생성/소멸시 자동 호출
3. **this 포인터**: 객체 자신을 가리키는 포인터
4. **메소드 체이닝**: `*this` 반환으로 연속 호출 가능
5. **클래스 배열**: 디폴트 생성자 필요
6. **참조 반환**: 객체 자체를 반환하여 연속 사용 가능
