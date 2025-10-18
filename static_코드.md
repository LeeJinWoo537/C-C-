# C++ Static 키워드와 복사 생성자

## 얕은 복사 (Shallow Copy)

### 개념
얕은 복사는 객체의 값만 복사하는 방식입니다. 만약 객체가 포인터를 가지고 있다면, 그 포인터의 주소만 복사하게 됩니다. 즉, 두 객체가 같은 메모리 공간을 참조하게 되므로, 한 객체가 그 메모리를 수정하거나 삭제하면 다른 객체에도 영향을 미칩니다.

### 얕은 복사 예제

```cpp
class Person
{
public:
    Person(int id, const char* nmae);
    ~Person();
    void changeName(const char* name);
    void show();

private:
    char* name;
    int id;
};

Person::Person(int id, const char* name)
{
    this->id = id;
    int len = strlen(name);
    this->name = new char[len + 1];
    strcpy(this->name, name);
}

Person::~Person()
{
    if (name)
    {
        delete[] name;
    }
    cout << "시스템이 종료되었습니다." << endl;
}

void Person::changeName(const char* name)
{
    if (strlen(name) > strlen(this->name))
    {
        return;
    }
    strcpy(this->name, name);
}

void Person::show()
{
    cout << id << ", " << name << endl;
}

int main()
{
    Person father(1, "Kitae");
    Person daughter(father);

    cout << "daughter 객체 생성 직후 ----" << endl;

    father.show();
    daughter.show();

    daughter.changeName("Grace");
    cout << "daughter 이름을 Grace로 변경한 후 ----" << endl;

    father.show();
    daughter.show();

    return 0;
}
```

### 얕은 복사의 문제점
1. **데이터 공유 문제**: 두 객체가 같은 메모리 공간을 가리키기 때문에, 하나의 객체에서 데이터를 변경하면 다른 객체에 영향을 미칩니다.
2. **메모리 해제 문제**: 두 객체가 동일한 메모리를 가리키고 있기 때문에, 하나의 객체가 소멸될 때 해당 메모리를 삭제하면, 다른 객체는 이미 삭제된 메모리를 참조하게 되어 문제가 발생할 수 있습니다.

## 깊은 복사 (Deep Copy)

### 개념
깊은 복사는 객체를 복사할 때 객체의 데이터뿐만 아니라, 데이터가 참조하는 메모리까지 복사하는 방식입니다. 즉, 새로운 메모리를 할당하고, 그 메모리 공간에 원본 객체의 데이터를 복사하는 방식입니다.

### 깊은 복사 예제

```cpp
class Person
{
public:
    Person(int id, const char* nmae);
    Person(const Person& person);  // 복사 생성자
    ~Person();
    void changeName(const char* name);
    void show();

private:
    char* name;
    int id;
};

Person::Person(int id, const char* name)
{
    this->id = id;
    int len = strlen(name);
    this->name = new char[len + 1];
    strcpy(this->name, name);
}

Person::Person(const Person& person)
{
    this->id = person.id;
    int len = strlen(person.name);
    this->name = new char[len + 1];  // 새로운 메모리 할당
    strcpy(this->name, person.name);  // 데이터 복사
    cout << "복사 생성자 실행. 원본 객체의 이름" << this->name << endl;
}

Person::~Person()
{
    if (name)
    {
        delete[] name;
    }
    cout << "시스템이 종료되었습니다." << endl;
}

void Person::changeName(const char* name)
{
    if (strlen(name) > strlen(this->name))
    {
        return;
    }
    strcpy(this->name, name);
}

void Person::show()
{
    cout << id << ", " << name << endl;
}

int main()
{
    Person father(1, "Kitae");
    Person daughter(father);

    cout << "daughter 객체 생성 직후 ----" << endl;

    father.show();
    daughter.show();

    daughter.changeName("Grace");
    cout << "daughter 이름을 Grace로 변경한 후 ----" << endl;
    
    father.show();
    daughter.show();

    return 0;
}
```

### 깊은 복사의 장점
- 각각 독립적인 메모리를 갖게 되어, 한 객체의 변경이나 삭제가 다른 객체에 영향을 미치지 않습니다.
- 메모리 안전성이 보장됩니다.

## 디폴트 매개변수 (Default Parameters)

### 기본 사용법

```cpp
void star(int a = 5);
void msg(int id, string text = "");

void star(int a)
{
    for (int i = 0; i < a; i++)
    {
        cout << '*';
    }
    cout << endl;
}

void msg(int id, string text)
{
    cout << id << ' ' << text << endl;
}

int main()
{
    star();        // 기본값 5 사용
    star(10);      // 10 사용

    msg(10);       // 기본값 "" 사용
    msg(10, "Hello");  // "Hello" 사용

    return 0;
}
```

### 여러 매개변수가 있는 경우

```cpp
void f(char c = ' ', int line = 1);

void f(char c, int line)
{
    for (int i = 0; i < line; i++)
    {
        for (int j = 0; j < 10; j++)
        {
            cout << c;
        }
        cout << endl;
    }
}

int main()
{
    f();           // 기본값 사용
    f('%');        // c만 변경
    f('@', 5);     // 둘 다 변경

    return 0;
}
```

### 클래스에서의 디폴트 매개변수

```cpp
void fillLine(int n = 25, char c = '*')
{
    for (int i = 0; i < n; i++)
    {
        cout << c;
    }
    cout << endl;
}

int main()
{
    fillLine();        // 기본값 사용
    fillLine(10, '%'); // 둘 다 변경

    return 0;
}
```

### 클래스 생성자에서의 디폴트 매개변수

```cpp
class MyVector
{
public:
    MyVector(int n = 100);  // 디폴트 매개변수
    ~MyVector();

private:
    int* p;
    int size;
};

MyVector::MyVector(int n)
{
    p = new int[n];
    size = n;
}

MyVector::~MyVector()
{
    delete[] p;
}

int main()
{
    MyVector* v1, * v2;
    v1 = new MyVector();      // 기본값 100 사용
    v2 = new MyVector(1024);   // 1024 사용

    delete v1;
    delete v2;

    return 0;
}
```

## 함수 오버로딩과 모호성

### 모호한 호출 예제

```cpp
float square(float a)
{
    return a * a;
}

double square(double a)
{
    return a * a;
}

int main()
{
    cout << square(3.0) << endl;
    // cout << square(3) << endl;   // 에러: 중복된 함수에 대한 모호한 호출
    cout << square((float)3);       // 타입 캐스팅으로 해결

    return 0;
}
```

### 참조 매개변수에서의 모호성

```cpp
int add(int a, int b)
{
    return a + b;
}

int add(int a, int& b)
{
    b = b + a;
    return b;
}

int main()
{
    int s = 10, t = 20;
    // cout << add(s, t);  // 매개 변수에 대한 모호성

    return 0;
}
```

### 디폴트 매개변수에서의 모호성

```cpp
void msg(int id)
{
    cout << id << endl;
}

void msg(int id, string s = "")
{
    cout << id << ":" << s << endl;
}

int main()
{
    msg(5, "Good Morning");
    // msg(6);   // 모호함: 어떤 함수를 호출할지 불분명

    return 0;
}
```

## Static 키워드

### Static 멤버 함수

```cpp
class Math
{
public:
    Math();
    ~Math();
    static int abs(int a)
    {
        return a > 0 ? a : -a;
    }
    static int max(int a, int b)
    {
        return (a > b) ? a : b;
    }
    static int min(int a, int b)
    {
        return (a > b) ? b : a;
    }

private:

};

Math::Math()
{
}

Math::~Math()
{
    cout << "시스템이 종료되었습니다." << endl;
}

int main()
{
    cout << Math::abs(-5) << endl;
    cout << Math::max(10, 8) << endl;
    cout << Math::min(-3, -8) << endl;

    return 0;
}
```

**Static 멤버 함수의 특징:**
- 객체를 생성하지 않고도 클래스 이름으로 직접 호출 가능
- 객체를 생성하는 것이 아니라 객체 안에 있는 함수만 호출하는 것이므로 소멸자가 호출되지 않음

### Static 멤버 변수

```cpp
class Circle
{
public:
    Circle(int r = 1);
    ~Circle();
    double getArea();
    static int getNumOfCircles();

private:
    static int numOfCircles;  // static 멤버 변수
    int radius;
};

Circle::Circle(int r)
{
    radius = r;
    numOfCircles++;  // 객체 생성시 카운트 증가
}

Circle::~Circle()
{
    numOfCircles--;  // 객체 소멸시 카운트 감소
    cout << "시스템이 종료되었습니다." << endl;
}

double Circle::getArea()
{
    return 3.14 * radius * radius;
}

int Circle::getNumOfCircles()
{
    return numOfCircles;
}

int Circle::numOfCircles = 0;  // static 멤버 변수 초기화

int main()
{
    Circle* p = new Circle[10];
    cout << "생존하고 있는 원의 개수 = " << Circle::getNumOfCircles() << endl;

    delete[] p;
    cout << "생존하고 있는 원의 개수 = " << Circle::getNumOfCircles() << endl;

    Circle a;
    cout << "생존하고 있는 원의 개수 = " << Circle::getNumOfCircles() << endl;

    Circle b;
    cout << "생존하고 있는 원의 개수 = " << Circle::getNumOfCircles() << endl;

    return 0;
}
```

**Static 멤버 변수의 특징:**
- 클래스의 모든 객체가 공유하는 변수
- 클래스 외부에서 초기화해야 함
- 객체 생성/소멸과 함께 카운트 관리 가능

## 주요 포인트

1. **얕은 복사 vs 깊은 복사**:
   - 얕은 복사: 포인터만 복사하여 메모리 공유
   - 깊은 복사: 새로운 메모리 할당하여 독립적인 복사

2. **디폴트 매개변수**:
   - 오른쪽 매개변수부터 기본값 설정
   - 함수 호출시 생략 가능한 매개변수 제공

3. **함수 오버로딩**:
   - 매개변수 타입이나 개수로 구분
   - 모호한 호출 방지 필요

4. **Static 키워드**:
   - Static 멤버 함수: 객체 생성 없이 호출 가능
   - Static 멤버 변수: 모든 객체가 공유하는 변수
   - 클래스 외부에서 초기화 필요
