# 항목51: new 및 delete를 작성할 때 따라야 할 기존의 관례를 잘 알아 두자 - 작성자: 신동욱

<aside>

# 🔎 뭐 할거냐?

operator new를 짜는 관례를 소개하는거임

operator delete 짜는 관례도 있음

근데 관례라고 했는데 거의 룰임

이거 지키는게 좋은듯?

</aside>

---

## 기존 관례에 잘 맞는 operator new

### 1. 반환값의 일관성 유지

- 요청된 크기만큼의 메모리를 마련해 줄 수 있으면
    
    → 그 메모리에 대한 포인터를 반환
    
- 마련해줄 수 없으면
    
    → `std::bad_alloc` 예외를 던져야 함
    
    > `operator new` 는 `malloc` 처럼 `NULL`을 반환하면 안됨
     C++에서는 실패 시 반드시 예외를 던지는 것이 규칙
    > 

### 2. 가용 메모리가 부족할 때의 처리 (new-handler 호출)

- C++에는 `new_handler`라는 전역 함수 포인터가 있음
    
    이 함수는 메모리 부족 시 호출되어 복구를 시도함
    
- `operator new` 는 메모리 확보 실패 시 이 핸들러를 호출해야 함

### 3. 0바이트 요청에 대한 처리

- 크기가 없는(0바이트) 메모리 요청에 대한 대비책을 갖춰야 한다
    - 0바이트가 요구되었을 때조차도 적법한 포인터를 반환해야 한다
    - 이로 인해 다른 부분들이 좀 간단해진다

### 4. 기본 형태의 `operator new`를 가리지 않기

- 사용자가 정의한 `operator new`가 C++ 표준의 기본 버전을 덮어쓰면 안됨
- 즉, 클래스 전용 버전을 만들더라도 전역 new/delete와 호환되도록 정의해야 함

### 비멤버 버전의 `operator new` 함수 의사 코드

```cpp
void* operator new(std::size_t size) throw(std::bad_alloc)
{
	using namespace std;

	if (size == 0) {
		size = 1;
	}

	while (true) {
		// size바이트를 할당해 봅니다
		if (할당이 성공했음)
			return (할당된 메모리에 대한 포인터);

		// 할당이 실패했을 경우, 현재의 new 처리자 함수가
		// 어느 것으로 설정되어 있는지 찾아냅니다.
		new_handler globalHandler = set_new_handler(0);
		set_new_handler(globalHandler);

		if (globalHandler) (*globalHandler) ();
		else throw std::bad_alloc();
	}
}
```

1. 외부에서 0바이트를 요구했을 때 1바이트 요구인 것으로 간주
2. `operator new`가 메모리 할당에 실패했을 때, 현재 등록된 new-handler 함수가 있으면 호출해서 복구 기회를 주고, 없으면 `std::bad_alloc` 예외를 던진다
    1. 근데 문제는 C++ 표준에는 “현재 handler를 조회하는 함수”가 없다
    2. 그래서 set_new_handler를 이용한 트릭을 사용
        1. `new_handler globalHandler = set_new_handler(0);` ⇒ 현재 handler를 새 handler로 바꾸고, 기존 handler를 반환
        2. 원복하기 위해 `set_new_handler(globalHandler)` 

- 단일 스레드 환경에서 잘 동작
- 다중 스레드 환경에서는 뮤텍스 락이 필요

## operator new 함수에는 무한 루프가 들어있다

```cpp
while (true) {
    if (메모리 할당 성공) return 포인터;
    // 실패 시 new-handler 호출
}
```

- 위 코드를 보면 무한 루프 `while(true)`가 떡하니 있다
- 이 루프를 빠져나오는 유일한 조건은
    - 메모리 할당이 성공하거나,
    - 항목 49에서 이야기한 동작들 중 하나를 new handler 함수 쪽에서 해주거나
        1. 가용 메모리 늘려주기: 캐시를 비우거나, 다른 메모리 영역을 해제해서 malloc이 성공하도록 유도
        2. 다른 new-handler 설치: 다른 처리자를 등록하여 다른 전략을 수행
        3. new 처리자의 설치를 제거하든가: 더 이상 복구 시도 없이 루프를 빠져나가기 위한 준비
        4. bad_alloc 혹은 bad_alloc에서 파생된 타입의 예외를 던지든가: 루프 종료
        5. 아예 함수 복귀를 포기하고 도중 중단하든가: 프로세스 중단

## 주의! operator new 멤버 함수는 파생 클래스 쪽으로 상속된다

### 상황

- 특정 클래스 전용의 할당자를 만들어서 할당 효율을 최적화하기 위해서 사용자 정의 메모리 관리자를 작성할 수 있다
- 근데 특정 클래스란 ‘그’ 클래스 하나를 가리켜야 한다
    - 그니까 어떤 X라는 클래스를 위한 `operator new` 함수가 있으면,
    - 이 함수의 동작은 크기가 `sizeof(X)`인 객체에 맞춰져 있다는 말

### 문제

- 상속때문에 파생 클래스 객체를 담을 메모리를 할당하는 데 기본 클래스의 `operator new` 함수가 호출되는 일이 벌어진다

```cpp
class Base {
public:
	static void* operator new(std::size_t size) throw(std::bad_alloc);
	...
};

class Derived: public Base	// Derived에서는 operator new가
{... };						// 선언되지 않았다

Derived* p = new Derived;	// Base::operator new가 호출되네!
```

![image.png](attachment:8d0906f5-e218-4a09-a92f-f8888b8f1019:image.png)

### 해결 방법

- 크기 확인해서 표준 new를 호출하도록

```cpp
void* Base::operator new(std::size_t size) throw(std::bad_alloc) {
	if (size != sizeof(Base))					// "틀린 크기가 들어오면,
		return ::operator new(size);		// 표준 operator new 쪽에서 메모리
												            // 할당 요구를 처리하도록 넘긴다

	...											          // 맞는 크기가 들어오면 메모리 할당
												            // 요구를 여기서 처리한다
}												
```

- 잠깐잠깐!
- 0바이트 상황을 점검하지 않았다고요옷!
- size ≠ sizeof(Base)가 사실은 0바이트 점검을 포함하고 있다
- 왜냐? C++에서는 모든 독립 구조의 객체는 반드시 크기가 0이 넘어야 하는 금기 사항이 있기 때문

## 배열에 대한 메모리 할당을 클래스 전용 방식으로 하고 싶다면, operator new[] 함수를 구현하면 된다

### `operator new[]`의 역할

- `operator new[]` 안에서 해 줄 일은 단순히 원시 메모리의 덩어리를 할당하는 것밖에 없다
- 배열에 실제로 들어갈 객체들은 아직 **생성 전**이므로,
    
    → 배열 안에 몇 개의 객체가 들어갈지 알 수 없음
    
- 따라서 `operator new[]` 안에서는 배열 원소 수나 객체 크기 계산을 정확히 할 수 없음

### 상속과 크기 문제

```cpp
Derived* arr = new Derived[10]; // Derived는 Base 상속
```

- `Derived`가 Base보다 크면,
    
    → `Base::operator new[]`가 호출될 경우, `size`가 실제 배열에 필요한 메모리보다 더 작거나 크게 계산될 수 있음
    
- 따라서 `Base::operator new[]` 에서는 배열 원소 하나의 크기를 sizeof(Base)로 가정할 수 없음

### size 인자의 의미

- `operator new[]`에 전달되는 `size`는 **배열 원소들의 실제 크기 x 개수 + 헤더 공간**일 수 있음
- C++에서 동적 배열은 배열 원소 수를 저장할 헤더 공간이 추가되므로,
    
    → `size`가 단순히 `n x sizeof(T)`와 같지 않음
    
- 따라서 `operator new[]` 내부에서는 단순히 `size`만큼 원시 메모리를 확보하고 반환하는 정도로만 처리

---

## operator delete에 대한 관례

- C++은 널 포인터에 대한 `delete`적용이 항상 안전하도록 보장한다는 사실만 잊지 않으면 만사 OK 입니다~
- 여러분이 할 일은 이 보장을 유지하는 것

### 비멤버 버전 `operator delete`

```cpp
void operator delete(void* rawMemory) throw()
{
	if (rawMemory == 0) return;			// 널 포인터가 delete되려고 할 경우에는
										// 아무것도 하지 않게 한다

	rawMemory가 가리키는 메모리를 해제한다

}
```

### 클래스 전용 버전 `operator delete`

```cpp
class Base {
public:
	static void* operator new(std::size_t size) throw(std::bad_alloc);
	static void* operator delete(void* rawMemory, std::size_t size) throw();
	...
};

void operator delete(void* rawMemory, std::size_t size) throw()
{
	if (rawMemory == 0) return;			// 널 포인터에 대한 점검
	
	if (size != sizeof(Base)) {         // 크기가 "틀린" 경우,
		::operator delete(rawMemory);   // 표준 operator delete가
		return;							// 메모리 삭제 요청을 맡도록 한다
	}

	rawMemory가 가리키는 메모리를 해제한다

	return;
}
```

- 클래스 전용 operator delete 역시 “틀린 크기로 할당된” 메모리의 삭제 요청을 ::operator delete 쪽으로 전달하는 식으로 구현

---

## 마지막으로 재밌는(?)이야기

- 가상 소멸자가 없는 기본 클래스로부터 파생된 클래스의 객체를 삭제하려고 할 경우에는 operator delete로 C++가 넘기는 size_t값이 엉터리일 수 있다
- 그래서 C++에서 기본 클래스에 가상 소멸자(virtual destructor)가 반드시 필요

### 1. 기본 전제

C++에서 `delete` 는 두 단계를 거친다

1️⃣ **소멸자 호출**

2️⃣**`operator delete` 호출**(즉, 메모리 해제)

즉,

```cpp
delete p;
```

를 실행하면 실제로는 이렇게 된다

```cpp
p->~ClassName();           // 1. 소멸자 호출
operator delete(p, size);  // 2. 메모리 해제
```

여기서 `size`는 `sizeof(ClassName)` 값

---

### 2. 문제 상황: 가상 소멸자가 없는 기본 클래스

```cpp
class Base {
public:
    Base() { std::cout << "Base ctor\n"; }
    ~Base() { std::cout << "Base dtor\n"; }
};

class Derived : public Base {
public:
    Derived() { std::cout << "Derived ctor\n"; }
    ~Derived() { std::cout << "Derived dtor\n"; }
};

int main() {
    Base* p = new Derived;
    delete p;  // ⚠️ 문제 발생!
}
```

- `p`의 타입은 `Base*` 이다
- 따라서 `delete p;` 할 때 컴파일러는 “Base 객체를 지운다”고 생각함

그 결과:

1. `Base::~Base()`만 호출되고
2. `Derived::~Derived()`는 전혀 호출되지 않는다
3. 그리고 더 심각하게, `operator delete(p, sizeof(Base))`가 호출됨

즉, C++은 이 객체의 실제 크기가 Derived의 크기인지 모른다

그래서 **엉뚱한 `size_t` 값 (`sizeof(Base)`)**을 `operator delete`에 넘겨줌

---

### 3. 왜 위험하냐

클래스가 커스텀 `operator delete`를 정의하고 그 안에서 `size`를 이용한다면,

잘못된 크기 정보로 메모리를 해제하거나 메모리 풀을 관리하게 된다

---

### 4. 해결책 : `virtual` 소멸자

```cpp
class Base{
public:
	virtual ~Base() { std::cout << "Base dtor\n"; }
};
```

이렇게 하면 `delete p;` 수행 시:

1. 런타임에 **객체의 실제 타입(=Derived)**을 확인해서
2. Derived의 소멸자를 먼저 호출하고,
3. 그 후에 Base의 소멸자를 호출한다
4. 마지막으로 **올바른 크기(`sizeof(Derived)`)** 가 `operator delete`로 전달된다

즉, 잘못된 타입 인식으로 인해 객체의 실제 크기를 모른 채 잘못된 크기 정보로 메모리를 해제할 수 있다

그래서 “기본 클래스에는 반드시 `virtual` 소멸자를 두어야 한다”는 거임