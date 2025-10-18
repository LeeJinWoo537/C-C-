# C++ new와 delete 사용법

## 개요
C++에서 동적 메모리 할당을 위한 `new`와 `delete` 연산자의 사용법을 설명합니다.

## 기본 개념
- `new`를 사용할 때는 변수가 포인터 변수여야 함
- `delete`를 사용할 때는 `delete[]` 형태로 사용해야 함

## 기본 사용법

### 1. 배열 동적 할당
```cpp
double* p3 = new double[3];

p3[0] = 0.2;
p3[1] = 0.5;
p3[2] = 0.8;

cout << "p3[1] is " << p3[1] << ".\n";

p3 = p3 + 1;    // p3++ == p3 + 1

cout << "Now p3[0] is " << p3[0] << " and ";
cout << "p3[1] is " << p3[1] << ".\n";

p3 = p3 - 1;
delete[] p3;     // 메모리 동적할당을 해제할 때는 delete[] p3; 형태로 사용
```

**주의사항:**
- `delete p3[];` 형태로는 사용할 수 없음
- 해제할 때 배열 `[]` 사이즈(크기)를 적으면 안됨

### 2. 문자열 동적 할당 예제
```cpp
#define SIZE 20

using namespace std;

int main() {
    char animal[SIZE];    // 먼저 배열 사이즈를 생성했기 때문에 new로 메모리 동적을 못함
    char* ps;

    cout << "동물 이름을 입력하십시오.\n";
    cin >> animal;       // 입력 값

    ps = new char[strlen(animal) + 1];  // 5 + 1 = 6 char형 타입에 배열 사이즈 6이라는 것을 포인터 변수 *ps에게 배열 사이즈를 주겠다라는 뜻
    strcpy(ps, animal);                        // 복사

    cout << "입력하신 동물 이름을 복사하였습니다" << endl;
    cout << "입력하신 동물 이름은 " << animal << "이고 그 주소는 " << (int*)animal << " 입니다. " << endl;    // (int*) == 주소 값
    cout << "복사된 동물 이름은 " << ps << "이고, 그 주소는 " << (int*)ps << " 입니다." << endl;

    delete[] ps;
    
    return 0;
}
```

### 3. 구조체 동적 할당
```cpp
struct st {
    char name[20];           // 20바이트
    int age;                 // 4바이트
};

int main() {
    st* temp = new st;       // 구조체를 메모리 동적, 24바이트

    cout << "당신의 이름을 입력하십시오.\n";
    cin >> temp -> name;

    cout << "당신의 나이를 입력하십시오.\n";
    cin >> (*temp).age;        // temp->age == (*temp).age
    
    cout << "안녕하세요! " << temp->name << "씨!\n";
    cout << "당신은 " << temp -> age << "살 이군요!.";

    delete temp;
    
    return 0;
}
```

## 주요 포인트
1. **포인터 변수 사용**: `new`는 항상 포인터 변수에 할당
2. **메모리 해제**: `delete[]`를 사용하여 배열 메모리 해제
3. **구조체 접근**: `->` 연산자 또는 `(*ptr).member` 형태로 접근
4. **문자열 길이**: `strlen() + 1`로 NULL 문자 공간 확보
