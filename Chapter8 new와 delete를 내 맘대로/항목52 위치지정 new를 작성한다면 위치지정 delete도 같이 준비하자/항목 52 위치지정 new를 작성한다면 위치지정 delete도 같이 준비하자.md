# 항목 52: 위치지정 new를 작성한다면 위치지정 delete도 같이 준비하자 - 작성자: 고형주

<aside>

# 🔎 이것만은 잊지 말자!

- `operator new` 함수의 위치지정(placement) 버전을 만들 때는, 이 함수와 짝을 이루는 위치지정 버전의 `operator delete` 함수도 꼭 만들어주세요. 이 일을 빼먹었다가는, 찾아내기도 힘들며 또 생겼다가 안 생겼다 하는 메모리 누출 현상을 경험하게 됩니다.
- `new` 및 `delete`의 위치지정 버전을 선언할 때는 의도한 바도 아닌데 이들의 표준 버전이 가려지는 일이 생기지 않도록 주의해주세요.
</aside>

---

<aside>

# 📌 기본 개념: new 표현식이 호출하는 두 개의 함수

</aside>

다음과 같은 new 표현식을 사용할 때 무슨 일이 일어나는지 알아보자.

```cpp
Widget *pw = new Widget;
```

## 호출되는 두 개의 함수

1. **`operator new`**: 메모리 할당
2. **`Widget` 생성자**: 객체 초기화

## 예외 발생 시나리오

만약 다음과 같은 상황이 발생한다면?

```cpp
Widget *pw = new Widget;
//              ↓
//         1. operator new 호출 → 성공 (메모리 할당 완료)
//         2. Widget 생성자 호출 → [예외 발생]
```

### 문제 상황

- 1단계의 메모리 할당은 이미 완료됨
- 2단계의 생성자에서 예외 발생
- `pw`에 포인터가 대입되지 않음 → 사용자 코드에서 메모리 해제 불가능
- **메모리 누출 발생**

1단계의 메모리 할당을 되돌리는 일은 C++ 런타임 시스템이 맡게 된다.

### C++ 런타임 시스템의 역할

**C++ 런타임 시스템이 1단계의 메모리 할당을 되돌려야 한다.**

이를 위해 런타임 시스템은

- 호출된 `operator new`와 짝이 되는 `operator delete`를 찾아서 호출

---

<aside>

# 📌 표준 형태의 new/delete

</aside>

## 기본형 `operator new`의 시그니처

```cpp
void* operator new(std::size_t) throw(std::bad_alloc);
```

## 기본형 `operator delete`의 시그니처

```cpp
// 전역 유효범위에서의 기본형 시그니처
void operator delete(void *rawMemory) throw();

// 클래스 유효범위에서의 전형적인 기본형 시그니처
void operator delete(void *rawMemory, std::size_t size) throw();
```

## 짝 맞추기는 간단하다

표준 형태의 `new` 및 `delete`만 사용하는 한,

- 런타임 시스템은 `new`의 동작을 되돌릴 `delete`를 쉽게 찾을 수 있음
- **별다른 문제 없음**

---

<aside>

# 📌 위치지정(Placement) new란?

</aside>

## 정의

**위치지정 `new`**: 기본형과 달리 **매개변수를 추가로 받는** 형태의 `operator new`

```cpp
// 추가 매개변수를 받는 operator new
void* operator new(std::size_t size, 추가_매개변수) throw(std::bad_alloc);
```

## 원조 위치지정 `new`

특히 유용한 형태: **객체를 생성시킬 메모리 위치를 나타내는 포인터**를 매개변수로 받는 것

```cpp
// "위치지정 new"의 원조 버전
void* operator new(std::size_t, void *pMemory) throw();
```

### 특징

- C++ 표준 라이브러리의 일부 (`<new>` 헤더에 포함)
- `vector`에서 미사용 공간에 원소 객체 생성 시 사용
- 가장 널리 알려진 위치지정 `new`

### 용어의 중의성

"위치지정 `new`"라는 용어는 두 가지 의미로 사용된다.

1. **좁은 의미**: `void*` 포인터를 추가로 받는 원조 버전
2. **넓은 의미**: 어떤 매개변수라도 추가로 받는 모든 `operator new`

문맥에 따라 의미를 파악해야 한다.

---

<aside>

# 📌 문제 상황: 위치지정 new와 메모리 누출

</aside>

## 예제: 로그 기록용 위치지정 `new`

메모리 할당 정보를 로그로 기록해줄 `ostream`을 지정 받음

```cpp
class Widget {
public:
	...
	// 비표준 형태의 operator new (로그 기록용)
	static void* operator new(std::size_t size, std::ostream& logStream) throw(std::bad_alloc);

	// 클래스 전용 operator delete의 표준 형태
	static void* operator delete(void *pMemory size_t size) throw();
	...
};
```

이는 문제가 있는 설계이다.

## 사용 예

```cpp
// operator new를 호출하면서 cerr을 ostream 인자로 넘김
// [문제] Widget 생성자에서 예외가 발생하면 메모리 누출 발생
Widget *pw = new (std::cerr) Widget;
```

`Widget` 객체 하나를 동적 할당할 때 `cerr`에 할당 정보를 로그로 기록하는 코드이다.

## 예외 발생 시 상황 분석

```cpp
Widget *pw = new (std::cerr) Widget;
//          ↓
//     1. operator new(size, std::cerr) 호출 → 성공
//     2. Widget 생성자 호출 → [예외 발생]
//     3. C++ 런타임 시스템이 메모리 할당을 되돌려야 함
//        → 어떤 operator delete를 호출해야 하나?
```

### 런타임 시스템의 딜레마

1. 호출된 `operator new`는 `ostream&` 타입의 매개변수를 추가로 받음
2. 이와 짝이 되는 `operator delete`를 찾아야 함
3. 즉, 다음과 같은 시그니처의 `operator delete`가 필요:

   ```cpp
   void operator delete(void *, std::ostream&) throw();
   ```

4. **하지만 위 `Widget` 클래스에는 이런 `operator delete`가 없음**
5. C++ 런타임 시스템은 혼란에 빠짐
6. **아무것도 하지 않음 → 메모리 누수 발생**

---

<aside>

# 📌 해결 방법: 위치지정 delete 제공하기

</aside>

## 규칙

추가 매개변수를 취하는 `operator new` 함수가 있는데, 그것과 **똑같은 추가 매개변수를 받는** `operator delete`가 짝으로 존재하지 않으면, 해당 `new`로 할당한 메모리를 해제해야 하는 상황이 와도 **어떤 `operator delete`도 호출되지 않는다.**

위의 코드에서 생길 수 있는 메모리 누출을 제거하려면, 로그 기록용 인자를 받는 위치지정 `new`와 짝이 되는 위치지정 `delete`를 `Widget` 클래스에 넣어주어야 한다는 결론이 나온다.

## 수정된 Widget 클래스

```cpp
class Widget {
public:
	...
	// 위치지정 new
	static void* operator new(std::size_t size, std::ostream& logStream)
		throw(std::bad_alloc);

	// 표준 형태의 operator delete (일반적인 delete 사용 시)
	static void operator delete(void *pMemory) throw();

	// [추가] 위치지정 new와 짝이 되는 위치지정 delete
	static void operator delete(void *pMemory, std::ostream& logStream) throw();
	...
};
```

## 이제 메모리 누수 문제가 해결됨

```cpp
// Widget 생성자에서 예외가 발생하더라도 메모리 누출 없음
Widget *pw = new (std::cerr) Widget;
```

### 예외 발생 시 동작

```cpp
Widget *pw = new (std::cerr) Widget;
//          ↓
//     1. operator new(size, std::cerr) 호출 → 성공
//     2. Widget 생성자 호출 → [예외 발생]
//     3. 런타임 시스템이 짝이 되는 operator delete(pMemory, std::cerr) 호출
//        → 메모리 할당 안전하게 되돌림 [문제 없음]
```

---

<aside>

# 📌 주의사항: 위치지정 delete는 언제 호출되나?

</aside>

## 두 가지 시나리오

### 시나리오 1: 생성자에서 예외 발생 (위치지정 `delete` 호출)

```cpp
Widget *pw = new (std::cerr) Widget;  // Widget 생성자에서 예외 발생
// → 위치지정 delete(void*, std::ostream&) 호출됨
```

### 시나리오 2: 일반적인 `delete` 사용 (기본 `delete` 호출)

```cpp
Widget *pw = new (std::cerr) Widget;  // 생성자 성공

// ...
// 사용
// ...

delete pw;  // → 기본 형태의 operator delete(void*) 호출됨
```

`Widget` 생성자가 예외를 던지지 않았고 사용자 코드의 `delete` 문까지 도달한 경우

## 정리

**위치지정 `delete`가 호출되는 유일한 경우**

- 위치지정 `new`의 호출에 '묶여서' 함께 호출되는 생성자에서 예외가 발생할 때뿐

**포인터에 `delete`를 적용했을 때**

- 절대로 위치지정 `delete`를 호출하지 않음
- 항상 표준 형태의 `operator delete`를 호출

## 메모리 누출 방지를 위한 요구사항

위치지정 `new`와 관련된 모든 메모리 누출을 사전에 봉쇄하려면,

1. **표준 형태의 `operator delete`** 제공
   - 객체 생성 도중에 예외가 던져지지 않았을 때를 대비
   - 일반적인 `delete` 사용 시 호출됨
2. **위치지정 `operator delete`** 제공
   - 위치지정 `new`와 똑같은 추가 매개변수를 받음
   - 생성자에서 예외가 던져졌을 때를 대비

이 두 가지 조건을 만족하면 메모리 누출이 생기지 않는다

```cpp
class Widget {
public:
    // 위치지정 new
    static void* operator new(std::size_t size, std::ostream& logStream)
        throw(std::bad_alloc);

    // [필수 1] 표준 형태의 delete (일반 delete pw; 용)
    static void operator delete(void *pMemory) throw();

    // [필수 2] 위치지정 delete (생성자 예외 발생 시)
    static void operator delete(void *pMemory, std::ostream& logStream) throw();
};
```

---

<aside>

# 📌 위험: 이름 가리기(Name Hiding) 문제

</aside>

## 문제 상황

C++의 이름 가리기 규칙

- 바깥쪽 유효범위의 함수와 클래스 멤버 함수의 **이름이 같으면,**
  바깥쪽 유효범위의 함수가 **이름만 같아도** 가려짐

## 예제 1: 클래스 전용 위치지정 new가 표준 new를 가림

```cpp
class Base {
public:
    ...
    // 이 new가 표준 형태의 전역 new를 가린다
    static void* operator new(std::size_t size, std::ostream& logStream)
        throw(std::bad_alloc);
    ...
};

// [에러] 표준 형태의 전역 operator new가 가려짐
Base *pb = new Base;

// [문제 없음] Base의 위치지정 new를 호출
Base *pb = new (std::cerr) Base;
```

개발자는 사용자 자신이 쓸 수 있다고 생각하는 다른 `new`들(표준 버전도 포함해서)을 클래스 전용의 `new`가 가리지 않도록 각별히 신경을 써야 한다.

### 왜 에러가 발생하는가?

- `Base` 클래스 안에 `operator new`가 선언됨
- 전역 유효범위의 표준 `operator new`가 가려짐
- `new Base`는 표준 형태를 찾지만, 가려져서 사용 불가

## 예제 2: 파생 클래스의 new가 기본 클래스의 new를 가림

```cpp
// 위의 Base로부터 상속받은 클래스
class Derived: public Base {
public:
	...
	// 기본형 new를 클래스 전용으로 다시 선언
	static void* operator new(std::size_t size) throw(std::bad_alloc);
	...
};

// [에러] Base의 위치지정 new가 가려져 있기 때문이다
Derived *pd = new (std::clog) Derived;

// [문제 없음] Derived의 operator new를 호출한다
Derived *pd = new Derived;
```

이런 기본 클래스를 이어받은 파생 클래스는 전역 `operator new`는 물론이고 자신이 상속 받은 기본 클래스의 `operator new`까지 가려버린다.

### 이름 가리기 계층

```cpp
전역 operator new (표준 형태)
    ↓ (가려짐)
Base::operator new (위치지정)
    ↓ (가려짐)
Derived::operator new (표준 형태)
```

---

<aside>

# 📌 표준 형태의 new/delete 정리

</aside>

## C++가 전역 유효범위에서 제공하는 세 가지 표준 형태

```cpp
// 1. 기본형 new
void* operator new(std::size_t size) throw(std::bad_alloc);

// 2. 위치지정 new (원조 버전)
void* operator new(std::size_t, void*) throw();

// 3. 예외불가 new (nothrow)
void* operator new(std::size_t, const std::nothrow_t&) throw();
```

## **[주의]** 이름 가리기의 영향

어떤 형태든 간에 `operator new`가 클래스 안에 선언되는 순간, 위의 **표준 형태들이 모두 가려진다.**

### 의도하지 않은 결과

```cpp
class MyClass {
public:
    // 사용자 정의 위치지정 new만 선언
    static void* operator new(std::size_t size, std::ostream& log)
        throw(std::bad_alloc);
};

// [에러] 기본형 new가 가려짐
MyClass *p1 = new MyClass;

// [에러] 원조 위치지정 new가 가려짐
void* buffer = malloc(sizeof(MyClass));
MyClass *p2 = new (buffer) MyClass;

// [에러] nothrow new가 가려짐
MyClass *p3 = new (std::nothrow) MyClass;

// [문제 없음] 사용자 정의 위치지정 new만 사용 가능
MyClass *p4 = new (std::cerr) MyClass;
```

---

<aside>

# 📌 해결 방법: 표준 형태도 사용 가능하게 만들기

</aside>

## 원칙

사용자가 표준 형태를 쓰지 못하게 막을 의도가 아니었다면,

- 사용자 정의 `operator new` 외에 **표준 형태들도 사용자가 접근할 수 있도록** 해야 함

물론 `operator new`를 만들었다면,

- **`operator delete`도 함께** 만들어야 함

클래스의 울타리 안에서 이런저런 할당/해제 함수들이 여느 때와 똑같은 방식으로 동작했으면 하는 경우에는, 그냥 클래스 전용 버전이 전역 버전을 호출하도록 구현해두면 된다.

## 방법 1: 각각 구현하기

```cpp
class Widget {
public:
    // 표준 형태들을 모두 제공
    static void* operator new(std::size_t size) throw(std::bad_alloc);
    static void operator delete(void *pMemory) throw();

    static void* operator new(std::size_t size, void *ptr) throw();
    static void operator delete(void *pMemory, void *ptr) throw();

    static void* operator new(std::size_t size, const std::nothrow_t&) throw();
    static void operator delete(void *pMemory, const std::nothrow_t&) throw();

    // 사용자 정의 형태
    static void* operator new(std::size_t size, std::ostream& log) throw(std::bad_alloc);
    static void operator delete(void *pMemory, std::ostream& log) throw();
};
```

### 문제점

- 코드 중복
- 유지보수 어려움
- 실수하기 쉬움

그래서 다음의 권장 방법을 사용한다.

---

<aside>

# 📌 권장 방법:

StandardNewDeleteForms 기본 클래스 사용

</aside>

## 1단계: 표준 형태를 모두 갖춘 기본 클래스 작성

```cpp
class StandardNewDeleteForms {
public:
	// 1. 기본형 new/delete
	static void* operator new(std::size_t size) throw(std::bad_alloc)
	{ return ::operator new(size); }

	static void operator delete(void *pMemory) throw()
	{ ::operator delete(pMemory); }

	// 2. 위치지정 new/delete (원조 버전)
	static void* operator new(std::size_t size, void *ptr) throw()
	{ return ::operator new(size, ptr); }

	static void operator delete(void *pMemory, void *ptr) throw()
	{ return ::operator delete(pMemory, ptr); }

	// 3. 예외불가 new/delete (nothrow)
	static void* operator new(std::size_t size, const std::nothrow_t& nt) throw()
	{ return ::operator new(size, nt); }

	static void operator delete(void *pMemory, const std::nothrow_t& nt) throw()
	{ ::operator delete(pMemory); }
}
```

기본 클래스 하나를 만들고, 이 안에 `new` 및 `delete`의 기본 형태를 전부 넣어두면 된다.

### 구현 방식

- 각 클래스 전용 버전이 **전역 버전을 호출**하도록 구현
- `::operator new`는 전역 `operator new`를 의미

```cpp
// StandardNewDeleteForms를 상속받음 (표준 형태를 물려받음)
class Widget: public StandardNewDeleteForms {
public:
	// 표준 형태가 (Widget 내부에) 보이도록 만듦
	using StandardNewDeleteForms::operator new;
	using StandardNewDeleteForms::operator delete;

	// 사용자 정의 위치지정 new를 추가
	static void* operator new(std::size_t size, std::ostream& logStream)
		throw(std::bad_alloc);

	// 앞의 것과 짝이 되는 위치지정 delete를 추가
	static void operator delete(void *pMemory, std::ostream& logStream) throw();
	...
};
```

## 동작 방식

### using 선언의 역할

```cpp
using StandardNewDeleteForms::operator new;
using StandardNewDeleteForms::operator delete;
```

- 기본 클래스의 `operator new/delete`를 파생 클래스의 유효범위로 끌어옴
- 이름 가리기 문제 해결

표준 형태에 덧붙여 사용자 정의 형태를 추가하고 싶다면, 이제는 이 기본 클래스를 축으로 넓혀 가면 된다. 상속과 `using` 선언을 2단 콤보로 사용해서 표준 형태를 파생 클래스 쪽으로 끌어와 외부에서 사용할 수 있게 만든 후에, 원하는 사용자 정의 형태를 선언해야 한다.

### 사용 예

```cpp
Widget w1;                                  // 스택 객체

Widget *pw1 = new Widget;                   // [문제 없음] 기본형 new
Widget *pw2 = new (buffer) Widget;          // [문제 없음] 위치지정 new (원조)
Widget *pw3 = new (std::nothrow) Widget;    // [문제 없음] nothrow new
Widget *pw4 = new (std::cerr) Widget;       // [문제 없음] 사용자 정의 위치지정 new

delete pw1;                                 // [문제 없음] 기본형 delete

// pw2는 위치지정 new로 생성 → 명시적 소멸자 호출 필요
pw2->~Widget();
delete pw3;                                 // [문제 없음] 기본형 delete
delete pw4;                                 // [문제 없음] 기본형 delete
```

---

<aside>

# 📌 전체 구조 정리

</aside>

## new/delete 짝 맞추기 규칙

| **상황**               | **호출되는 delete**                  | **비고**                       |
| ---------------------- | ------------------------------------ | ------------------------------ |
| `new Widget`           | `operator delete(void*)`             | 표준 형태                      |
| `new (ptr) Widget`     | `operator delete(void*, void*)`      | 원조 위치지정 (생성자 예외 시) |
| `new (nothrow) Widget` | `operator delete(void*, nothrow_t&)` | nothrow (생성자 예외 시)       |
| `new (log) Widget`     | `operator delete(void*, ostream&)`   | 사용자 정의 (생성자 예외 시)   |
| `delete pw`            | `operator delete(void*)`             | 항상 표준 형태                 |

## 흐름도

```cpp
new 표현식 실행
    ↓
1. operator new 호출 (메모리 할당)
    ↓ [성공]
2. 생성자 호출
    ↓
    ├─→ [성공] → 객체 생성 완료
    │
    └─→ [예외 발생]
        ↓
        런타임 시스템이 operator new와 같은 시그니처의
        operator delete를 찾아서 호출 (메모리 할당 되돌림)
```

## 구현 체크리스트

**사용자 정의 위치지정 `new`를 만들 때,**

1. 같은 시그니처의 위치지정 `delete`도 함께 제공
2. 표준 형태의 `delete`도 제공 (일반 `delete` 사용 시)
3. 다른 표준 형태들도 가려지지 않도록 조치

**`StandardNewDeleteForms` 패턴 사용 시,**

1. 기본 클래스를 상속
2. `using` 선언으로 표준 형태 노출
3. 사용자 정의 `new`/`delete` 쌍 추가

---

<aside>

# 📌 개념 정리

</aside>

### 1. 위치지정 new란?

**추가 매개변수를 받는 모든 `operator new`**

- 넓은 의미: 어떤 매개변수든 추가로 받는 `operator new`
- 좁은 의미: `void*` 포인터를 추가로 받는 원조 버전

### 2. 위치지정 delete란?

**위치지정 new와 같은 추가 매개변수를 받는 `operator delete`**

- 생성자에서 예외가 발생했을 때만 호출됨
- 일반 `delete` 사용 시에는 호출되지 않음

### 3. 왜 위치지정 delete가 필요한가?

**메모리 누출을 방지하기 위해**

```cpp
new (log) Widget
      ↓
operator new(size, log) 호출 → 성공
Widget 생성자 호출 → 예외 발생
      ↓
[상황 1. 위치지정 delete 없음]
    → 메모리 누출

[상황 2. 위치지정 delete 있음]
    → operator delete(pMemory, log) 호출
    → 메모리 해제
```

### 4. 이름 가리기는 왜 문제인가?

**의도하지 않게 표준 형태를 사용할 수 없게 됨**

- 클래스에 하나의 `operator new`만 선언해도, 모든 표준 형태의 `operator new`가 가려짐
- 사용자가 기본형 `new`조차 사용할 수 없음

### 5. using 선언은 어떻게 도움이 되는가?

**기본 클래스의 함수를 파생 클래스 유효범위로 끌어옴**

```cpp
class Widget: public StandardNewDeleteForms {
public:
    // 기본 클래스의 모든 operator new를 Widget 내부로 가져옴
    using StandardNewDeleteForms::operator new;
    using StandardNewDeleteForms::operator delete;

    // 추가로 사용자 정의 형태 선언
    static void* operator new(std::size_t, std::ostream&);
    static void operator delete(void*, std::ostream&);
};
```

---

<aside>

# 📌 정리 예제

</aside>

## 잘못된 예제

```cpp
class Widget {
public:
    // [문제 1] 위치지정 delete가 없음
    static void* operator new(std::size_t size, std::ostream& log) throw(std::bad_alloc);

    static void operator delete(void *pMemory) throw();

    // [문제 2] 표준 형태들이 가려짐
};

// [메모리 누출 위험] 생성자에서 예외 발생 시
BadWidget *pbw = new (std::cerr) BadWidget;

// [에러] 기본형 new가 가려짐
BadWidget *pbw2 = new BadWidget;
```

## 올바른 예제

```cpp
class Widget: public StandardNewDeleteForms {
public:
    // [해결 1] 표준 형태 노출
    using StandardNewDeleteForms::operator new;
    using StandardNewDeleteForms::operator delete;

    // [해결 2] 위치지정 new와 delete 쌍으로 제공
    static void* operator new(std::size_t size, std::ostream& log) throw(std::bad_alloc);

    static void operator delete(void *pMemory, std::ostream& log) throw();
};

// [문제 없음] 생성자에서 예외 발생해도 메모리 누출 없음
GoodWidget *pgw = new (std::cerr) GoodWidget;

// [문제 없음] 기본형 new 사용 가능
GoodWidget *pgw2 = new GoodWidget;

// [문제 없음] 모든 표준 형태 사용 가능
GoodWidget *pgw3 = new (std::nothrow) GoodWidget;
```

---

<aside>

# 📌 결론

</aside>

## 원칙

1. **위치지정 `new`를 작성하면 위치지정 `delete`도 작성하라**
   - 생성자 예외 발생 시 메모리 누출 방지
2. **표준 형태의 `delete`도 제공하라**
   - 일반적인 `delete` 사용 시를 위해
3. **이름 가리기를 조심하라**
   - 의도하지 않게 표준 형태를 막지 말 것
   - `using` 선언이나 `StandardNewDeleteForms` 패턴 활용
4. **일관성을 유지하라**
   - `new`와 `delete`는 항상 쌍으로
   - 추가 매개변수는 동일하게

## 정리

<aside>

**`new`가 있으면 `delete`가 있어야 한다.**

**위치지정 `new`가 있으면 위치지정 `delete`가 있어야 한다.**

**둘 다 없으면 표준 형태를 가리지 말아야 한다.**

</aside>

---
