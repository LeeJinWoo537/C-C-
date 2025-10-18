# 모던 C++ 스마트 포인터

## 1. 스마트 포인터

C++에서 동적 메모리 관리와 관련된 여러 문제를 해결해주는 중요한 도구입니다. 기본적으로 메모리를 명시적으로 new와 delete로 관리하는 대신 

스마트 포인터가 객체의 생애주기를 자동으로 관리해주기 때문에 메모리 누수나 더블 프리 등의 문제를 예방할 수 있습니다.

## 2. 스마트 포인터 종류

- **unique_ptr**: 하나의 객체에 대한 소유권을 명확히 하고, 복사할 수 없도록 할 때.
- **shared_ptr**: 여러 객체가 동일한 리소스를 공유해야 할 때.
- **weak_ptr**: (shared_ptr)와 연관된 객체를 약하게 참조해야 할 때.

3개의 스마트 포인터를 사용하려면 `#include <memory>` 라이브러리를 사용해야 함

## 스마트 포인터 장점

- **자동 메모리 관리**: 스마트 포인터는 객체의 생애주기를 자동으로 관리하여 메모리 누수 방지.

- **메모리 안전성**: unique_ptr와 shared_ptr는 객체 소유권을 명확히 하여 메모리 해제의 시점을 제어합니다.

- **예외 안전성**: 예외가 발생해도 스마트 포인터가 객체의 메모리를 자동으로 관리하므로 예외 발생 시에도 안전하게 메모리를 해제할 수 있습니다.

## unique_ptr

- unique_ptr은 소유권이 하나의 객체에만 존재하도록 보장하는 스마트 포인터
- 한 unique_ptr이 소유한 메모리는 다른 unique_ptr에 의해 소유될 수 없고 복사할 수 없습니다.
- 객체가 unique_ptr의 범위를 벗어나면 자동으로 메모리가 해제됩니다.
- `std::unique_ptr<int>` 이건 스마트 포인터 객체 타입이라고 생각하면 됨 unique_ptr은 포인터 객체라서 int 값을 직접 저장을 하는게 아니라 동적 메모리를 가리키는 역할을 해

### 기본 사용법
```cpp
#include <iostream>
#include <memory>

int main() {
    // unique_ptr를 생성하고 메모리 할당
    std::unique_ptr<int> ptr1 = std::make_unique<int>(10);

    // ptr1의 소유권을 ptr2로 이동 (복사 불가)
    std::unique_ptr<int> ptr2 = std::move(ptr1);

    // ptr1은 이제 null pointer 상태
    if (!ptr1) {
        std::cout << "ptr1 is now null" << std::endl;           // 출력값: ptr1 is now null
    }

    std::cout << "ptr2 points to: " << *ptr2 << std::endl;   // 출력 값: ptr2 points to: 10

    return 0;
}
```

메모리는 ptr2가 소멸될 때 자동으로 해제됩니다.

unique_ptr은 따로 경로를 따지지 않아 이 스마트 포인터는 포인터의 주소값 자체를 관리하는 것이 아니라 오직 소유권을 관리하는 스마트 포인터
`std::unique_ptr<int> ptr1;` 이런식으로 그냥 생성해도 괜찮아 그니까 값이나 경로를 따로 없이 이렇게 생성이 가능해

여기서 소유권이라는 개념은 동적 할당된 메모리를 관리하는 권한을 의미해
어떤 unique_ptr 객체가 특정 메모리를 관리하는지 여부를 뜻해

`std::make_unique<int>;` 이건 new와 같은 메모리 동적할당이다. 그래서 이건 unique_ptr 변수일 때만 사용할 수 있어 사실상 전용 함수라고 생각하면 돼

```cpp
int* p = std::make_unique<int>(10);  // ❌ 오류 발생!
```

뒤에 있는 (10)은 값을 이야기하는거다 그래서 출력하면 10이 나온다

### 소유권 개념
여기서 나오는 소유권은 변수 하나당 하나의 소유권을 가진다는 말이다
즉 하나의 std::unique_ptr 객체가 관리하는 메모리는 오직 그 unique_ptr만이 소유할 수 있다.

어라? 그러면 unique_ptr<int> 이거 2개 생성하면 동작이 안되는거야? 라고 할 수 있지만

**예시:**
```cpp
std::unique_ptr<int> ptr1 = std::make_unique<int>(10);
std::unique_ptr<int> ptr2 = std::make_unique<int>(20);
```

그렇지 않고 이런식으로 코드를 짜도 정상적으로 동작해 왜냐면 각각 새로운 동적 메모리를 생성했기 때문이야 각각 따로 동적 메모리를 생성한거지 

```cpp
std::unique_ptr<int> ptr1 = std::make_unique<int>(10);
std::unique_ptr<int> ptr2 = std::make_unique<int>(20);

ptr2 = std::move(ptr1);  // ptr1에서 ptr2로 이동!
```

이런식으로도 동작이 가능해 ptr1에 메모리가 ptr2로 간거지 이렇게하면 기존에 있던 ptr2에가 가리키던 20은 더이상 접근할 방법이 없음므로 자동으로 해제되고 ptr1이 가지고 있던 10이 ptr2로 이동하게 돼

- ptr1 -> nullptr
- ptr2 -> 10

근데 오류가 발생하는 경우는

**예시:**
```cpp
std::unique_ptr<int> ptr1 = std::make_unique<int>(10);
std::unique_ptr<int> ptr2 = ptr1;  // ❌ 복사하려고 하면 오류 발생!
```

ptr1과 ptr2가 동일한 메모리를 공유하려고 하면 오류가 발생
std::unique_ptr은 동일한 메모리를 여러 개를 공유할 수 없기 때문에 오류가 생겨

대신에 소유권은 이동할 수 있어

### 소유권을 이동한다는 의미
소유권을 이동한다는 의미가 약간 헷갈릴 수 있는데
예를 들어서

```cpp
std::unique_ptr<int> ptr1 = std::make_unique<int>(10);
std::unique_ptr<int> ptr2 = std::move(ptr1);
```

위에있는 이 코드를 예로 들면 ptr1의 소유권을 ptr2로 이동한다 이건 단순히 값만 이동하는 것이 아니라 동적 메모리에 대한 권한이 이동하는 거랑 같아

쉽게 말하면 ptr1이 관리하던 힙 메모리의 주소를 ptr2가 가지게 된거라고 생각하면 돼
이 과정에서 ptr1은 이제 그 메모리를 관리하지 않으므로 nullptr이 되고 ptr2만이 10을 관리하게 될 수 있는거야

그러면 ptr1의 메모리는 그대로 있어? 라고 물어보면
메모리는 그대로 있지만 포인터 ptr1은 그 메모리르 관린할 권한을 잃어 그래서 `ptr1.get();`을 출력하면 nullptr이 나와 `*ptr1`을 출력하면 런타임 오류가 발생할 수 있어

그렇다고 nullptr이 됐다고 해서 `ptr1 = 10;` 이런식으로 일반 변수로 변한건 아니야 아직도 unique_ptr 타입에 스마트 포인터야

그래서 다시 힙 메모리를 생성해서 사용할 수 있어

```cpp
std::unique_ptr<int> ptr1 = std::make_unique<int>(10);
std::unique_ptr<int> ptr2 = std::move(ptr1);  // ptr1 → nullptr, ptr2 → 10

ptr1 = std::make_unique<int>(20);  // 새로운 힙 메모리 할당!
```

## 문제 해결

### 문제1
std::unique_ptr을 사용하여 int 배열을 동적으로 할당합니다. (배열 크기는 5로 설정)
배열에 1부터 5까지의 값을 넣고 출력하는 프로그램을 작성하세요.
std::unique_ptr의 소유권을 다른 포인터로 이동시키고, 이동 후 첫 번째 원소의 값을 변경한 후, 두 포인터를 통해 배열의 값을 출력하세요.
이동 후 원본 포인터는 nullptr이 되어야 하며, 이동 후의 포인터를 사용해 값을 출력해야 합니다.

**출력 값:** 
```
배열의 값: 1 2 3 4 5
배열의 값 (소유권 이동 후): 10 2 3 4 5
```

**내가 푼 문제:**
```cpp
#include <iostream>
#include <memory>

using namespace std;

int main() {
    unique_ptr<int> ptr[5] = make_unique(0);

    for (int i = 0; i < 5; i++) {
       *ptr[i] = i + 1; 
    }
    
    unique_ptr<int> ptr2 = move(ptr);

    for (int i = 0; i < 5; i++) {
        cout << ptr2[i] << endl;
    }
    
    return 0;
}
```

배열로 만들때는

```cpp
#include <iostream>
#include <memory>

using namespace std;

int main() {
    // 크기가 5인 unique_ptr 배열 생성
    unique_ptr<int[]> ptr = make_unique<int[]>(5);

    // 배열 초기화
    for (int i = 0; i < 5; i++) {
        ptr[i] = i + 1;
        cout << ptr[i] << " ";
    }

    // unique_ptr은 이동 가능!
    unique_ptr<int[]> ptr2 = move(ptr);

    cout << endl;

    // 이동 후 ptr은 nullptr이 됨
    if (!ptr) {
        cout << "ptr은 더 이상 소유권이 없습니다!" << endl;
    }
    
    ptr2[0] = 10;    // ptr[0] = 10 이렇게 하면 ptr2가 출력이 안됨 nullptr이기 때문에 값을 주더라도 못받음 그리고 nullptr이기 때문에 출력에 문제가 생김

    // 이동된 ptr2에서 값 출력
    for (int i = 0; i < 5; i++) {
        cout << ptr2[i] << " ";
    }

    return 0;
}
```

`<int[]>` 이런식으로 타입을 넣는 공간에 배열 표시를 한다

### 문제2

- std::unique_ptr을 사용하여 struct 타입을 동적으로 할당합니다.
- struct는 int x와 int y 멤버 변수를 가지고 있고, 생성자에서 x와 y 값을 초기화합니다.
- unique_ptr을 통해 객체를 동적으로 할당하고, 그 값을 출력한 후, 값을 변경한 뒤 다시 출력하세요.
- 소유권을 이동시킨 후에도 값이 유지되는지 확인하고, nullptr로 이동한 포인터를 확인하세요.

**출력 값:**
```
객체 값: 5 10
객체 값 (소유권 이동 후): 20 10
```

```cpp
#include <iostream>
#include <memory>

using namespace std;

struct myst {
    int x;
    int y;
};

int main() {
    unique_ptr<myst> ptr1 = make_unique<myst>(5, 10);

    cout << *ptr1->x << " " << *ptr1->y << endl;

    unique_ptr<myst> ptr2 = move(ptr1);

    ptr2->x += 15;

    cout << *ptr2->x << " " << *ptr2->y << endl;
        
    return 0;
}
```

**내가 푼 코드**
근데 오류가 걸림
이유는??

`make_unique<myst>(5, 10);` 여기 부분에서 오류가 걸림 왜냐면 생성자를 만들어야 값을 줄 수 있는데 구조체는 그러한 생성자가 없어서 생성자를 만들고 값을 따로 줘야해
그래서

```cpp
unique_ptr<myst> ptr1 = make_unique<myst>(); 
ptr1->x = 5;
ptr1->y = 10;
```

이런식으로 하는게 좋아 
애초부터 `make_unique<>();` 이건 힙 메모리 객체를 생성해주는거라 객체를 생성하면 당연히 생성자도 생성되고 하니까

약간 코드로 보면 이해하기가 쉬울 수 있는데

```cpp
#include <iostream>
#include <memory>
using namespace std;

struct myst {
    int x, y;
    myst() {   // 기본 생성자
        cout << "myst 생성자 호출!" << endl;
        x = 0;
        y = 0;
    }
};

int main() {
    auto ptr1 = make_unique<myst>();  // 생성자 호출됨!
    auto ptr2 = make_unique<int>();   // 기본 자료형은 생성자 없음, 그냥 0으로 초기화

    return 0;
}
```

이런식으로 생성된다고 보면 돼

`make_unique<int>();` 이건 기본 자료형이 힙 메모리에 저장된다고 생각하면 돼 근데 다만 클래스처럼 생성자가 있는게 아니라 그냥 메모리를 할당하고 값을 저장하는 것 뿐이야

그리고 약간 다른 이야기긴 한데

```cpp
int* ptr1;  // 정수형 포인터 선언 (아직 아무것도 가리키지 않음)
unique_ptr<int> ptr2;  // 스마트 포인터 선언 (아직 아무것도 가리키지 않음)
```

이거 두 개는 거의 같은 의미야 아무것도 가리키지 않는 포인터라고 생각하면 돼

그리고 오류걸리는게 하나 더 있는데 바로 `*ptr1->x` 이거야 이건 int 값을 역참조 한거라 이렇게 출력하면 오류가 걸려 이미 ptr1은 (->) 이걸로 x 값을 가리키고 있는데 포인터(*)를 사용하면 x에 타입인 int 타입(자료형)을 출력하는거라 오류가 걸려

여기서 역참조는 가리키는걸 의미해

이것도 쉽게 코드로 알려주자면

```cpp
int a = 5;
int* p = &a;
cout << *p << endl;
```

이건 가리키는게 포인터(*) 밖에 없으니까 그냥 p는 a에 주소 값을 가리킴
근데 *p는 a에 값을 가리키고 그래서 정상적으로 5가 출력이 됨

근데 구조체는

```cpp
struct myst {
    int x, y;
};

int main() {
    myst obj = { 1, 2 };
    myst* p = &obj;

    cout << p->x << endl;  // O (x 값을 출력)
    cout << *p->x << endl;  // ❌ (이미 x는 int인데, 또 역참조할 필요 없음)
}
```

위에 있는 코드 처럼 이미 -> 이걸로 값을 가리키고 있기 때문에 굳이 포인터(*)를 안써도 출력이 가능함

그래서 정확히 출력을 하면

```cpp
#include <iostream>
#include <memory>

using namespace std;

struct myst {
    int x;
    int y;
};

int main() {
    unique_ptr<myst> ptr1 = make_unique<myst>();

    ptr1->x = 5;
    ptr1->y = 10;

    cout << ptr1->x << " " << ptr1->y << endl;

    unique_ptr<myst> ptr2 = move(ptr1);

    if (!ptr1) {
        cout << "ptr1은 더 이상 소유권이 없습니다!" << endl;
    }

    ptr2->x += 15;

    cout << ptr2->x << " " << ptr2->y << endl;
        
    return 0;
}
```

이렇게 코드를 하면 돼

이건 좀 다른 이야긴 한데 동적할당을 new 사용하면

```cpp
myst* p = new myst();  // 힙에 객체 생성
p->x = 5;
p->y = 10;
delete p; // 메모리 해제 필요
```

이런식으로 사용해야해서 

근데 unique_ptr은

```cpp
unique_ptr<myst> ptr = make_unique<myst>(); // 힙에 객체 생성, 자동 메모리 관리
ptr->x = 5;
ptr->y = 10;
```

이렇게만 사용해도 되서 편리함

## shared_ptr

- 여러 개의 포인터가 하나의 객체를 공유할 수 있도록 해주는 스마트 포인터
- 같은 객체를 여러 개의 shared_ptr이 가리킬 수 있고 마지막 shared_ptr이 소멸될 때 객체가 삭제 돼

### 1. 참조 카운트
- shared_ptr은 객체를 몇 개의 shared_ptr이 공유하고 있는지 참조 카운트를 관리해
- shared_ptr이 새로운 곳에 복사되면 참조 카운트가 1 증가하고, 소멸되면 1 감소해
- 참조 카운트가 0이 되면 객체가 자동으로 삭제 돼

### 2. 객체 공유 가능
- unique_ptr은 소유권을 하나의 포인터만 가지지만, shared_ptr은 여러 개의 포인터가 소유할 수 있음

### 3. std::make_shared< >() 사용
- make_shared를 쓰면 더 효율적으로 객체를 생성할 수 있음
- 동적 할당을 한 번만 수행해서 성능이 더 좋음

### 예시 코드
```cpp
#include <iostream>
#include <memory>

int main() {
    // shared_ptr 생성
    std::shared_ptr<int> ptr1 = std::make_shared<int>(20);

    // ptr2는 ptr1을 공유
    std::shared_ptr<int> ptr2 = ptr1;

    std::cout << "ptr1 points to: " << *ptr1 << std::endl;
    std::cout << "ptr2 points to: " << *ptr2 << std::endl;

    // 참조 카운트 확인
    std::cout << "Reference count: " << ptr1.use_count() << std::endl;

    return 0;
}
```

### 참조 카운트 관리
```cpp
#include <iostream>
#include <memory>

using namespace std;

int main() {
    shared_ptr<int> ptr1 = make_shared<int>(10); // 10을 가리키는 shared_ptr 생성

    shared_ptr<int> ptr2 = ptr1;  // ptr1을 복사 (참조 카운트 증가)

    cout << "ptr1: " << *ptr1 << ", ptr2: " << *ptr2 << endl; // 10 10
    cout << "참조 카운트: " << ptr1.use_count() << endl;  // 2

    ptr1.reset();  // ptr1이 소유권을 포기 (참조 카운트 감소)

    cout << "ptr2: " << *ptr2 << endl; // ptr2는 여전히 객체를 가리킴
    cout << "참조 카운트: " << ptr2.use_count() << endl; // 1

    return 0;
}
```

### 참조 카운트 설명
1. 참조 카운터는 객체를 공유할 때 1이 증가가 되는데 객체를 생성할 때 부터 참조 카운트 1이 증가 돼

```cpp
shared_ptr<int> ptr1 = make_shared<int>(10);  // 참조 카운트 1
shared_ptr<int> ptr2 = ptr1;  // ptr2는 ptr1을 복사하여 객체를 공유. 참조 카운트 2
```

그리고 `make_shared<int>` 이거는 shared_ptr일 때만 사용이 가능해!
객체를 동적으로 할당할 때 한 번에 메모리를 살당해서 성능으로 최적화하는 방법이기 때문이야.

2. 참조 카운트가 필요한 이유는 메모리 관리를 자동으로 해주는 역할을 해서야
여러 개의 shared_ptr이 동일한 객체를 공유하고 있다면 마지막 shared_ptr이 소멸할 때 객체가 자동으로 삭제 되게끔 하는데 이를 위해 참조 카운트가 필요해

자동 메모리 관리
메모리 누수 방지 등의 장점이 있어

### 사용 예시
동적 객체 관리: 여러 개의 컴포넌트가 동일한 객체를 사용해야 할 때 유용해. 예를 들어, 여러 모듈에서 하나의 객체를 사용해야 할 때, 이 객체가 더 이상 사용되지 않으면 자동으로 메모리를 해제해줘.
자원 관리: 자원을 공유하는 경우에 메모리 관리의 복잡함을 줄일 수 있어.

대신에 참조 카운트가 많아지면 메모리 사용량이 늘어날 수 있지만 그 자체로 성능에 큰 영향을 주지는 않아 참조 카운트를 갱신하는데 드는 비용이 들 수 있어

성능 문제: shared_ptr은 참조 카운트를 관리하기 위해 원자적(atomic) 연산을 사용해. 따라서 참조 카운트가 많아지면, 동기화나 멀티스레드 환경에서 조금 더 많은 오버헤드가 발생할 수 있어.

### 참조 카운트 관련 함수
참조 카운트를 출력할 때는
```cpp
ptr1.use_count()
```

소유권 포기할 때는
```cpp
ptr1.reset();
```

## weak_ptr

- weak_ptr은 shared_ptr과 함께 사용하는 스마트 포인터로 객체의 소유권을 가지지 않으면서도 객체를 참조할 수 있는 포인터

### weak_ptr이 필요한 이유
shared_ptr은 참조 카운트를 증가시키는데 순환 참조 문제를 일으킬 수 있음 근데 weak_ptr은 객체를 참조하지만 참조 카운트를 증가시키지 않아서 순환 참조를 방지하는 데 사용 돼
shared_ptr끼리 순환 참조가 많아지면 메모리 누수가 발생해
이유는 참조 카운트가 0이 되지 않아서 객체가 소멸되지 않기 때문이야.

### 기본 weak_ptr 예제
```cpp
#include <iostream>
#include <memory>

using namespace std;

int main() {
    shared_ptr<int> sp = make_shared<int>(10);
    weak_ptr<int> wp = sp;  // weak_ptr은 shared_ptr을 참조하지만 소유권은 가지지 않음

    cout << "shared_ptr use_count: " << sp.use_count() << endl; // 1
    cout << "weak_ptr use_count: " << wp.use_count() << endl;   // 1 (shared_ptr의 count를 보여줄 뿐, 증가시키지는 않음)

    return 0;
}
```

### weak_ptr의 expired() 와 lock()
- **expired()**: 참조 대상이 삭제 되었는지 확인하는 함수(참조 대상이 사라지면 true)
- **lock()**: shared_ptr을 생헝하여 안전하게 접근

### 예시
```cpp
#include <iostream>
#include <memory>

using namespace std;

int main() {
    weak_ptr<int> wp;

    {
        shared_ptr<int> sp = make_shared<int>(20);
        wp = sp;  // weak_ptr이 shared_ptr을 참조
        cout << "shared_ptr use_count: " << sp.use_count() << endl; // 1

        if (auto spt = wp.lock()) {  // lock()을 사용하여 shared_ptr 생성
            cout << "Value: " << *spt << endl; // 20
        }
    } // sp가 범위를 벗어나면서 메모리 해제됨

    if (wp.expired()) {
        cout << "The weak_ptr is expired!" << endl;  // 출력됨
    }

    return 0;
}
```

lock()을 사용하면 shared_ptr을 새로 만들어 안전하게 접근할 수 있음
expired()를 사용하면 참조하는 객체가 아직 존재하는지 확인할 수 있음

### lock()이 안전한 이유
weak_ptr을 직접 사용하면 객체가 이미 삭제되었는지 알 수 없어, 접근하면 미사용 메모리 접근 오류가 발생할 수 있음

lock()을 사용하면 객체가 살아 있을 때만 shared_ptr을 생성해서 안전하게 접근 가능 해

### 직접 접근하면 위험한 경우
```cpp
weak_ptr<int> wp;
{
    shared_ptr<int> sp = make_shared<int>(30);
    wp = sp;
} // sp가 사라짐 → 객체도 삭제됨

cout << *wp.lock() << endl; // 안전한 코드
cout << *wp.lock().get() << endl; // 위험한 코드! (nullptr 접근 가능성)
```

- lock()을 사용하지 않고 .get()을 바로 쓰면 이미 삭제된 메모리에 접근할 위험이 있음
- lock()을 사용하면 nullptr을 체크해서 안전하게 접근할 수 있음

### get()은 포인터 반환 함수
- get()은 shared_ptr이나 unique_ptr이 소유한 원래의 생 포인터를 반환하는 함수
- get()은 스마트 포인터가 내부적으로 관리하는 실제 포인터를 꺼내오는 역할

**예시:**
```cpp
shared_ptr<int> sp = make_shared<int>(100);
weak_ptr<int> wp = sp;

shared_ptr<int> temp = wp.lock(); // shared_ptr 생성
int* rawPtr = temp.get(); // get()으로 raw pointer 가져옴

cout << "Shared pointer value: " << *temp << endl; // 100 출력
cout << "Raw pointer value: " << *rawPtr << endl; // 100 출력
```

get()을 쓰면 스마트 포인터가 관리하는 원래 포인터를 얻을 수 있음
get()을 썼다고 해서 스마트 포인터가 객체의 소유권을 포기하는 건 아님
스마트 포인터 내부의 실제 포인터를 꺼내는 역할

### 출력할 때는 *포인터를 써서
```cpp
shared_ptr<int> sp = make_shared<int>(42);
weak_ptr<int> wp = sp; // weak_ptr이 sp를 참조

cout << *wp.lock() << endl; // 42 출력!
```

### 출력할 때 위험한 경우
```cpp
weak_ptr<int> wp;
{
    shared_ptr<int> sp = make_shared<int>(99);
    wp = sp;
} // sp가 소멸됨 (객체도 삭제됨)

cout << *wp.lock() << endl; // 위험! (널 포인터 역참조 가능성)
```

### 안전한 코드
```cpp
if (auto spt = wp.lock()) { // 객체가 살아 있으면 실행
    cout << *spt << endl;
} else {
    cout << "Pointer expired!" << endl;
}
```

`if(wp.lock())` 이런식으로 사용해도 괜찮지만 

```cpp
if (wp.lock()) {
    // 여기서 wp.lock()을 또 호출하면? 
    cout << *wp.lock() << endl;
}
```

이런식으로 사용하면 문제가 발생할 수 있음
shared_ptr이 생성되고 나서
또 `*wp.lock()`을 생성하면 새로운 shared_ptr이 또 생성돼
그러면 다른 곳에서 객체가 삭제되면 예상치 못한 동작이 생길 수 있음

### 참고로 lock()을 사용할 때 꼭 shared_ptr 변수를 만들 때는
```cpp
shared_ptr<int> sp = make_shared<int>(10);
weak_ptr<int> wp = sp;

shared_ptr<int> a = wp.lock(); // OK, shared_ptr 생성됨
```

이런식으로 해도 됨

### if (auto spt = wp.lock()) 설명
`if (auto spt = wp.lock())` 여기 부분에 보면 어떻게 이게 참이야? 왜냐면 if문은 참이여야 실행이 되잖아 근데 이건 어떻게해서 참이야?? 라고 할 수 있지만 

wp.lock()은 현재 weak_ptr이 가리키는 객체가 살아 있으면 shared_ptr을 생성해서 반환하고, 객체가 이미 삭제되었다면 빈 shared_ptr(nullptr)을 반환해!

shared_ptr이 nullptr이면 거짓(false)으로 판정되기 때문에 if문 안에서 참/거짓을 판단할 수 있음 반대로 객체가 살아있으면 참(true)

### lock() 객체가 살아 있는 경우
```cpp
shared_ptr<int> sp = make_shared<int>(10);
weak_ptr<int> wp = sp;  // weak_ptr이 sp를 참조

if (auto spt = wp.lock()) { // shared_ptr 생성 성공!
    cout << "Value: " << *spt << endl; // 10 출력
} else {
    cout << "Pointer expired!" << endl;
}
```

### lock() 객체가 삭제된 경우
```cpp
weak_ptr<int> wp;

{
    shared_ptr<int> sp = make_shared<int>(20);
    wp = sp;  // weak_ptr이 sp를 참조
} // sp가 범위를 벗어나면서 객체가 소멸됨

if (auto spt = wp.lock()) {     // shared_ptr = nullptr
    cout << "Value: " << *spt << endl;
} else {
    cout << "Pointer expired!" << endl; // 출력됨
}
```

## 순환 참조를 해결하는 weak_ptr 방법

### shared_ptr만 사용을 했을 때 발생될 수 있는 문제점
**예시:**
```cpp
#include <iostream>
#include <memory>

using namespace std;

struct Node {
    shared_ptr<Node> next;  // 서로 shared_ptr로 연결되어 있으면 메모리 해제가 안 됨!
    
    Node() { cout << "Node 생성" << endl; }
    ~Node() { cout << "Node 소멸" << endl; }
};

int main() {
    shared_ptr<Node> n1 = make_shared<Node>();
    shared_ptr<Node> n2 = make_shared<Node>();

    n1->next = n2;
    n2->next = n1;  // 서로 참조하면서 참조 카운트가 1에서 내려가지 않음 (메모리 누수 발생!)

    return 0;  // 프로그램이 끝나도 소멸자가 호출되지 않음! (메모리 누수 발생)
}
```

위에 있는 코드처럼 shared_ptr끼리 순환 참조를 하면 참조 카운트가 0이 되지 않아 메모리가 해제되지 않아

### 그치만 weak_ptr을 사용하면 순환 참조를 해결할 수 있어
**예시:**
```cpp
#include <iostream>
#include <memory>

using namespace std;

struct Node {
    weak_ptr<Node> next;  // weak_ptr을 사용하여 순환 참조 방지!

    Node() { cout << "Node 생성" << endl; }
    ~Node() { cout << "Node 소멸" << endl; }
};

int main() {
    shared_ptr<Node> n1 = make_shared<Node>();
    shared_ptr<Node> n2 = make_shared<Node>();

    n1->next = n2;  // weak_ptr이므로 참조 카운트 증가 X
    n2->next = n1;  

    return 0;  // 프로그램이 끝나면 정상적으로 소멸자가 호출됨
}
```

weak_ptr을 사용하면 shared_ptr의 참조 카운트가 증가하지 않아서 순환 참조 문제가 해결!

```
n1(shared_ptr) <--> next(weak_ptr) <--> n2(shared_ptr)
```

## 문제 해결

### weak_ptr 문제1
shared_ptr<int> 타입의 포인터를 하나 생성하고, 값 100을 저장한다.
이 shared_ptr을 weak_ptr로 참조하도록 한다.
weak_ptr을 lock()을 사용하여 shared_ptr로 변환한 후, 값을 출력한다.
원래의 shared_ptr을 reset()하여 삭제하고, weak_ptr이 가리키는 값이 존재하는지 확인한다.

**출력 값**
```
100  
해당 weak_ptr은 더 이상 유효하지 않습니다!  
```

```cpp
#include <iostream>
#include <memory>

using namespace std;

int main() {
    shared_ptr<int> sa = make_shared<int>(100);
    weak_ptr<int> wp = sa;

    shared_ptr<int> s1;
    if (s1 = wp.lock()) {
        cout << *s1 << endl;
    } else {
        cout << "해당 weak_ptr은 더 이상 유효하지 않습니다!" << endl;
    }

    sa.reset();  // 소유권 삭제

    if (wp.expired()) {
        cout << "객체가 살아 있습니다!" << endl;
    } else {
        cout << "해당 weak_ptr은 더 이상 유효하지 않습니다!" << endl;
    }

    return 0;
}
```

### 문제2
메모리 순환 참조를 해결하라

```cpp
#include <iostream>
#include <memory>

using namespace std;

class A;
class B;

class A {
public:
    shared_ptr<B> b;
    ~A() { cout << "A 소멸자" << endl; }
};

class B {
public:
    shared_ptr<A> a;
    ~B() { cout << "B 소멸자" << endl; }
};

int main() {
    shared_ptr<A> a = make_shared<A>();
    shared_ptr<B> b = make_shared<B>();

    a->b = b;  // A가 B를 참조
    b->a = a;  // B가 A를 참조

    return 0;
} 
```

**답:**
```cpp
#include <iostream>
#include <memory>

using namespace std;

class A;
class B;

class A {
public:
    //shared_ptr<B> b;
    weak_ptr<B> wp2;
    ~A() { cout << "A 소멸자" << endl; }
};

class B {
public:
    //shared_ptr<A> a;
    weak_ptr<A> wp1;
    ~B() { cout << "B 소멸자" << endl; }
};

int main() {
    shared_ptr<A> a = make_shared<A>();
    shared_ptr<B> b = make_shared<B>();

    a->wp2 = b;  // A가 B를 참조
    b->wp1 = a;  // B가 A를 참조

    return 0;
} 
```

### 문제3 (출력 값을 써라)
```cpp
#include <iostream>
#include <memory>
using namespace std;

void check_weak(weak_ptr<int> wp) {
    if (auto sp = wp.lock()) {
        cout << "값: " << *sp << endl;
    } else {
        cout << "객체가 삭제됨!" << endl;
    }
}

int main() {
    weak_ptr<int> wp;

    {
        auto sp = make_shared<int>(42);
        wp = sp; 
        check_weak(wp);  
    }

    check_weak(wp);  

    return 0;
}
```

**출력 값:**
```
42
객체가 삭제됨!
```

### 문제4 (순환 참조가 발생하는데 Node 객체들이 정상적으로 weak_ptr로 작동하게 하라)
```cpp
#include <iostream>
#include <memory>
using namespace std;

struct Node {
    int value;
    shared_ptr<Node> next;  

    Node(int v) : value(v) {
        cout << "Node 생성: " << value << endl;
    }
    ~Node() {
        cout << "Node 소멸: " << value << endl;
    }
};

int main() {
    auto node1 = make_shared<Node>(1);
    auto node2 = make_shared<Node>(2);

    node1->next = node2;
    node2->next = node1;  //  순환 참조 발생?

    return 0;
}
```

**weak_ptr로 바꾼 값:**
```cpp
#include <iostream>
#include <memory>
using namespace std;

struct Node {
    int value;
    //shared_ptr<Node> next;
    weak_ptr<Node> next;       // weak_ptr로 참조!

    Node(int v) : value(v) {
        cout << "Node 생성: " << value << endl;
    }
    ~Node() {
        cout << "Node 소멸: " << value << endl;
    }
};

int main() {
    auto node1 = make_shared<Node>(1);
    auto node2 = make_shared<Node>(2);

    node1->next = node2;
    node2->next = node1;  // 🔥 순환 참조 발생?

    return 0;
}
```

여기서 `Node(int v) : value(v)` 이거는 뭐냐?? 라고 물어볼 수 있는데 이거는

```cpp
Node(int v) {
    value = v;
}
```

이거를 쉽고 편하게 하기 위해서 `Node(int v) : value(v)` 이런식으로 작성 이걸 초기화 리스트라고 한다

### 초기화 리스트란?
멤버 변수를 더 효율적으로 초기화할 수 있는 방법입니다. 기본적으로 생성자 본문에서 멤버 변수를 초기화할 수 있지만, 초기화 리스트를 사용하면 멤버 변수들이 더 빨리 초기화될 수 있습니다.

`Node(int v) : value(v)` 이거랑  
```cpp
Node(int v) {
    value = v;
}
```

이거 두 개에 차이점은 
- `value = v;` 는 생성자 본문에서 실행됩니다. 즉, value가 객체가 생성된 후에 할당됩니다.
- `value(v)` 는 초기화 리스트에서 실행됩니다. 이 방법은 생성자 호출 시점에 value가 v로 초기화됩니다.

상속 관계나 초기화가 필요한 멤버 변수에 대해 더 많이 사용됩니다.

### 문제5 (출력 값을 써라)
```cpp
#include <iostream>
#include <memory>
using namespace std;

int main() {
    auto sp1 = make_shared<int>(100);
    weak_ptr<int> wp1 = sp1;

    cout << "초기 참조 카운트: " << sp1.use_count() << endl;

    {
        auto sp2 = wp1.lock();
        if (sp2) {
            cout << "sp2 값: " << *sp2 << endl;
            cout << "sp2 참조 카운트: " << sp2.use_count() << endl;
        }
    }

    cout << "스코프 종료 후 참조 카운트: " << sp1.use_count() << endl;
    return 0;
}
```

**출력 값:**
```
1
100
2
1
```

## reset 함수
`ptr.reset();` 또는 `ptr = nullptr;` 사용

## swap 함수
swap() 함수를 사용하면 두 스마트 포인터가 가리키는 객체를 교환할 수 있습니다. 이때 메모리 관리도 자동으로 처리됩니다.

**예:** `std::swap(ptr1, ptr2);`

### shared_ptr 활용 예시
```cpp
#include <iostream>
#include <memory>

class MyClass {
public:
    MyClass(int a) : data(a) {
        std::cout << "MyClass 객체 생성" << std::endl;
    }
    ~MyClass() {
        std::cout << "MyClass 객체 소멸" << std::endl;
    }
    void print() {
        std::cout << "데이터: " << data << std::endl;
    }
private:
    int data;
};

int main() {
    std::shared_ptr<MyClass> ptr1 = std::make_shared<MyClass>(100);
    std::cout << "ptr1가 가리키는 객체:" << std::endl;
    ptr1->print();

    std::shared_ptr<MyClass> ptr2 = ptr1;  // ptr2가 ptr1과 같은 객체를 가리킴
    std::cout << "ptr2가 가리키는 객체:" << std::endl;
    ptr2->print();

    std::cout << "객체 소멸은 ptr1, ptr2가 더 이상 가리키지 않을 때 이루어집니다." << std::endl;
}
```

## 스마트 포인터 비교표

| 스마트 포인터 | 소유권 | 참조 카운트 증가 | 용도 |
|---------------|--------|------------------|------|
| **unique_ptr** | 단독 | x | 하나의 스마트 포인터만 객체를 소유 이동만 가능 |
| **shared_ptr** | 공유 | o | 여러 개의 스마트 포인터가 하나의 객체를 공유 객체의 수명을 관리합니다. |
| **weak_ptr** | 비소유 | x | shared_ptr의 순환 참조 방지, 객체가 존재하는지 확인할 때 사용 |
