# C++ 학습 노트 - 객체 복사와 참조

## String 생성자와 초기화

### 기본 개념
```cpp
int main()
{
    string str;
    string address("서울시 성북구 삼선동 389");
    string copyAddress(address);

    char text[] = { 'L', 'o', 'v', 'e', ' ', 'C', '+', '+', '\0' };
    string title(text);

    cout << str << endl;
    cout << address << endl;
    cout << copyAddress << endl;
    cout << title << endl;

    return 0;
}
```

**`string title(text);`의 의미:**
- `title`은 `string` 타입의 변수
- `text`는 그 초기화 값으로 사용
- `text`는 C 스타일 문자열(char 배열)로, `string` 객체로 변환되어 `title`에 저장
- C++의 `string` 클래스는 C 스타일 문자열을 받아들이는 생성자를 제공

## 객체 복사와 값 전달

### Circle 클래스 예제
```cpp
class Circle
{
public:
    Circle();
    Circle(int radius);
    ~Circle();
    void setRadius(int r);
    int getRadius();
    double getArea();
private:
    int radius;
};

Circle::Circle()
{
    radius = 1;
    cout << "생성자 실행 radius = " << radius << endl;
}

Circle::Circle(int radius)
{
    this->radius = radius;
    cout << "생성자 실행 radius = " << radius << endl;
}

void Circle::setRadius(int radius)
{
    this->radius = radius;
}

int Circle::getRadius()
{
    return radius;
}

double Circle::getArea()
{
    return 3.14 * radius * radius;
}

Circle::~Circle()
{
    cout << "소멸자 실행 radius = " << radius << endl;
    cout << "시스템이 종료되었습니다." << endl;
}

void increase(Circle c)
{
    int r = c.getRadius();
    c.setRadius(r + 1);
}

int main()
{
    Circle waffle(30);
    increase(waffle);

    cout << waffle.getRadius() << endl;

    return 0;
}
```

### 출력 결과 분석
```
생성자 실행 radius = 30
소멸자 실행 radius = 31
시스템이 종료되었습니다.
30
소멸자 실행 radius = 30
시스템이 종료되었습니다.
```

**실행 흐름:**
1. `Circle waffle(30);` → 생성자 실행, radius = 30
2. `increase(waffle);` → 복사본 `c` 생성, 값 복사로 전달
3. `c.setRadius(r + 1);` → 복사본 `c`의 radius = 31로 변경
4. `increase()` 함수 종료 → 복사본 `c` 소멸, 소멸자 실행 (radius = 31)
5. `cout << waffle.getRadius();` → 원본 `waffle`의 radius = 30 출력
6. `main()` 종료 → 원본 `waffle` 소멸, 소멸자 실행 (radius = 30)

## 값 복사 vs 참조 전달

### 값 복사 (Value Copy)
```cpp
void increase(Circle c)  // 값 복사로 전달
{
    int r = c.getRadius();
    c.setRadius(r + 1);
}
```
- 객체의 복사본이 함수로 전달됨
- 원본 객체와 독립적인 메모리 공간
- 복사본의 변경이 원본에 영향을 미치지 않음

### 참조 전달 (Reference)
```cpp
void increase(Circle& c)  // 참조로 전달
{
    int r = c.getRadius();
    c.setRadius(r + 1);
}
```
- 원본 객체를 직접 참조
- 같은 메모리 공간을 공유
- 복사본의 변경이 원본에 직접 반영됨

## 얕은 복사 vs 깊은 복사

### 얕은 복사 (Shallow Copy)
- 포인터나 주소만 복사
- 원본과 복사본이 같은 메모리 공간을 공유
- 하나의 객체에서 값을 바꾸면 다른 객체도 영향받음
- 메모리 관리에 주의 필요

### 깊은 복사 (Deep Copy)
- 객체의 모든 데이터를 독립적으로 복사
- 원본과 복사본이 서로 다른 메모리 공간을 가짐
- 하나의 객체에서 값을 바꿔도 다른 객체에 영향 없음
- 메모리 안전성 보장

### 현재 코드의 경우
- 동적 메모리 할당이 없는 단순한 값 복사
- 얕은 복사도 깊은 복사도 아닌 "값 복사"
- 객체들이 독립적으로 동작하지만 메모리 참조 문제는 없음

## 객체 반환과 대입

### 함수에서 객체 반환
```cpp
Circle getCircle()
{
    Circle tmp(30);
    return tmp;
}

int main()
{
    Circle c;
    cout << c.getArea() << endl;

    c = getCircle();  // 객체 덮어쓰기
    cout << c.getArea() << endl;

    return 0;
}
```

**`c = getCircle();`의 동작:**
- `getCircle()`에서 `tmp` 객체 생성 (radius = 30)
- `return tmp;`에서 객체 전체가 반환됨 (값만이 아닌 객체 자체)
- 반환된 객체가 `c`에 할당되어 기존 객체를 덮어씀
- 객체 대입 연산자(Assignment Operator) 호출

## 참조와 얕은 복사의 유사성

### 참조 사용 예제
```cpp
int main()
{
    Circle circle;
    Circle& refc = circle;  // 참조
    refc.setRadius(10);
    cout << refc.getArea() << " " << circle.getArea() << endl;

    return 0;
}
```

**참조의 특징:**
- 객체의 별명(alias) 역할
- 원본 객체와 같은 메모리 공간을 공유
- 작동 방식은 얕은 복사와 유사하지만 기술적으로는 다름
- 메모리 주소를 직접 복사하지 않고 연결만 함

## Static 멤버 변수와 함수

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
    static int numOfCircles;    // 전역 변수
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
    cout << "시스템이 종료되었습니다." << ", " << "numofcircles에 값은: " << numOfCircles << endl;
}

int Circle::getNumOfCircles()
{
    return numOfCircles;
}

int Circle::numOfCircles = 0;  // 클래스 외부에서 초기화

int main()
{
    Circle* p = new Circle[10];
    cout << "생존하고 있는 원의 개수 = " << Circle::getNumOfCircles() << endl;

    delete[] p;   // 10개의 소멸자 실행
    cout << "생존하고 있는 원의 개수 = " << Circle::getNumOfCircles() << endl;   // 0

    Circle a;
    cout << "생존하고 있는 원의 개수 = " << Circle::getNumOfCircles() << endl;    // 1

    Circle b;
    cout << "생존하고 있는 원의 개수 = " << Circle::getNumOfCircles() << endl;    // 2

    return 0;
}
```

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
};

int main()
{
    cout << Math::abs(-5) << endl;
    cout << Math::max(10, 8) << endl;
    cout << Math::min(-3, -8) << endl;

    return 0;
}
```

**Static의 특징:**
- 객체 생성 없이 클래스 이름으로 직접 호출 가능
- 모든 객체가 공유하는 전역 변수/함수
- 객체 생성/소멸과 함께 카운트 관리 가능
- 소멸자가 호출되지 않음 (객체를 생성하지 않기 때문)

## Friend 함수

### Friend 함수 예제
```cpp
class Rect
{
public:
    Rect(int width, int height);
    ~Rect();
    friend bool equals(Rect r, Rect s);  // friend 함수 선언

private:
    int width;
    int height;
};

Rect::Rect(int width, int height)
{
    this->width = width;
    this->height = height;
}

Rect::~Rect()
{
    cout << "시스템이 종료되었습니다." << endl;
}

bool equals(Rect r, Rect s)
{
    if (r.width == s.height && r.height == s.height)  // private 멤버에 접근 가능
    {
        return true;
    }
    else
    {
        return false;
    }
}

int main()
{
    Rect a(3, 4); 
    Rect b(4, 5);

    if (equals(a, b))
    {
        cout << "equal" << endl;
    }
    else
    {
        cout << "not equal" << endl;
    }

    return 0;
}
```

**Friend 함수의 특징:**
- 클래스의 private 멤버에 접근 가능
- 클래스 외부에서 정의되지만 클래스 내부 멤버에 접근 권한을 가짐
- 캡슐화를 깨뜨리지만 특정 상황에서 유용

## 주요 포인트

1. **값 복사**: 객체의 복사본이 생성되어 독립적으로 동작
2. **참조 전달**: 원본 객체를 직접 참조하여 메모리 공간을 공유
3. **얕은 복사 vs 깊은 복사**: 메모리 참조 방식의 차이
4. **객체 반환**: 객체 전체가 반환되어 대입 연산자로 처리
5. **Static 멤버**: 모든 객체가 공유하는 전역 변수/함수
6. **Friend 함수**: private 멤버에 접근할 수 있는 특별한 함수
7. **메모리 관리**: 동적 할당이 없는 경우 값 복사가 안전하고 효율적
