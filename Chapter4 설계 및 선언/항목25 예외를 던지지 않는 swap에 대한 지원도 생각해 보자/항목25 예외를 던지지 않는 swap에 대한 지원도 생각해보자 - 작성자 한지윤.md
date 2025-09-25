# 항목25: 예외를 던지지 않는 swap에 대한 지원도 생각해보자 - 작성자: 한지윤

<aside>
🔎

# 이것만은 잊지말자!

- `std::swap` 이 여러분의 타입에 대해 느리게 동작할 여지가 있다면  `swap` 멤버 함수를 제공합시다. 이 멤버 `swap` 은 예외를 던지지 않도록 만듭시다.
- 멤버 swap을 제공했으면, 이 멤버를 호출하는 비멤버 `swap`도 제공합니다. 클래스(템플릿이 아닌)에 대해서는, `std::swap`도 특수화해 둡시다.
- 사용자 입장에서 `swap`을 호출할 때는, `std::swap`에 대한 `using` 선언을 넣어 준 후에 네임스페이스 한정 없이 `swap`을 호출합시다.
- 사용자 정의 타입에 대한 `std 템플릿`을 완전 특수화하는 것은 가능합니다. 그러나 `std`에 어떤 것 이라도 새로 ‘추가’하려고 들지는 마십시오.
</aside>

✅ swap은 단순한 함수 같지만 매우 중요합니다.

- `swap`은 두 객체의 값을 맞바꾸는 함수
- C++에서는 `STL`, 예외 안전성, 자기 대입 방지 등 다양한 맥락에서 자주 등장함
- 그래서 **“어떻게 제대로 구현하느냐”**가 중요해짐

## 🟩 기본 `swap` 함수

```cpp
template<typename T>
void swap(T& a, T& b) {
    T temp = a;
    a = b;
    b = temp;
}
```

위 코드는 표준 라이브러리에서 제공하는 기본  `swap`함수입니다.
복사 생성자와 복사 대입 연산자를 이용합니다.

### ⚡ 위 코드의 문제점?

- 세번의 복사가 발생하게 됩니다.
`a → temp` , `b → a` , `temp → b`
- 타입에 따라 이 복사가 매우 비쌀 수 있습니다. (성능 저하)
- 경우에 따라선 예외가 발생할 수도 있습니다.
예외가 발생하면 `swap`도중에 프로그램이 꼬일 수도…?

## 🟩 pImpl(pointer to implementation)

복사하면 손해를 보는 타입 중 으뜸을 꼽는다면
**다른 타입의 실제 데이터를 가리키는 포인터가 주성분인 타입**입니다.
대표적인게 `pImpl 관용구` 라고합니다~

아래는 `pImpl`설계를 차용한 `Widget`클래스

```cpp
class WidgetImpl {
    // 복사 비용이 엄청 높은 데이터
};

class Widget {
public:
		void swap(Widget& other)
		{
			using std::swap;  // 이거 왜쓰는지는 뒤에나옴
			swap(pImpl, other.pImpl) // 각 포인터를 맞바꿈
		}
private:
    WidgetImpl* pImpl;  // 포인터만 가짐
};
```

`Widget` 객체를 복사할 때 `pImpl` 포인터만 처리하면 되니 훨씬 가벼움

🔎 여기서 표준 `swap`을 사용하면 어떻게 될까?

- 표준 `std::swap`은 **복사 기반**입니다.
`WigetImpl` 은 **복사 비용이 엄청 높은 데이터**를 가지고 있습니다
- `Widget`에 대해 표준 `swap`을 사용하면 **엄청 비효율적임**

### ⚡ 효율적으로 하려면?

`Widget`타입에 대해 `std::swap`을 완전히 새로 정의하면 됩니다
→ 이걸 **특수화(specialize)** 라고 합니다

```cpp
namespace std {
    template<>
    void swap<Widget>(Widget& a, Widget& b) {
        swap(a.pImpl, b.pImpl);  // 포인터만 맞바꿈
    }
}
```

이건 기본 아이디어 코드입니다. 아직 컴파일이 안됩니다.

왜 안되냐면…….

```cpp
class WidgetImpl {
    // 복사 비용이 엄청 높은 데이터
};

class Widget {
public:
		// 코드...
private:
    WidgetImpl* pImpl;  // ⭐ 얘가 지금 private 입니다
};
```

```cpp
namespace std {
    template<>
    void swap<Widget>(Widget& a, Widget& b) {
        swap(a.pImpl, b.pImpl);  // 포인터만 맞바꿈
    }
}
```

- `pImpl`은 `private` 멤버이기 때문에 외부에서 접근할 수 없습니다
- 그래서 std::swap안에서 `a.pImpl`, `b.pImpl`을 사용할 수 없습니다

<aside>

- 완전 템플릿 특수화
    1. 템플릿 
        
        ```cpp
        template<typename T>
        void print(T value) {
            std::cout << value << std::endl;
        }
        ```
        
        어떤 타입이든 print 함수가 작동함 → 일반 템플릿
        
    
    1. 템플릿 특수화
        
        **특정 타입**에 대해 이 함수를 다르게 동작하도록 만들고 싶을 때 **특수화** 사용
        
        ```cpp
        // 일반 템플릿
        template<typename T>
        void print(T value) {
            std::cout << "일반 템플릿: " << value << std::endl;
        }
        
        // 완전 특수화 - T가 int일 때
        template<>
        void print<int>(int value) {
            std::cout << "int 특수화: " << value << std::endl;
        }
        ```
        
        그러면 이렇게 동작합니다
        
        ```cpp
        print(3.14);      // 일반 템플릿 호출 → "일반 템플릿: 3.14"
        print(10);        // 특수화된 템플릿 호출 → "int 특수화: 10"
        ```
        
    
    1. 언제씀?
        - 어떤 특정 타입에 대해서는 **일반적인 템플릿 동작이 비효율적이거나 맞지 않을 때**
        - 예외 없이 동작하게 하거나, 성능을 개선하기 위해 사용함
        
</aside>

### ⚡ 해결 하려면?

멤버함수로 `swap`을 만들만 됩니다.

```cpp
class Widget {
public:   // ⭐ 멤버함수로 만들어줌
    void swap(Widget& other) {
		    using std::swap; 
        swap(pImpl, other.pImpl);
    }
};
```

```cpp
namespace std {
    template<>
    void swap<Widget>(Widget& a, Widget& b) {
        a.swap(b);  // 멤버 함수 호출
    }
}
```

이렇게하면 컴파일도 되고, STL 컨테이너와 일관성도 유지됨

## 🟩 클래스 템플릿 이라면

### ⚡ 상황 가정: `Widget`이 템플릿 클래스일 때

```cpp
template<typename T>
class WidgetImpl { ... };

template<typename T>
class Widget { ... };
```

<aside>

- 😩 클래스 템플릿이 뭐임..
    
    하나의 클래스 정의로 여러 타입에 대해 재사용할 수 있도록 만든 C++의 기능
    
    > 클래스 안에서 사용할 데이터 타입을 미리 정하지 않고, 
    나중에 객체를 만들 때 타입을 지정하는 클래스
    > 
    
    **기본 문법**
    
    ```cpp
    template<typename T> // T는 타입 매개 변수(임시로 쓰는 타입 이름)
    class MyBox {
    public:
        T data;
    
        void set(T value) {
            data = value;
        }
    
        T get() {
            return data;
        }
    };
    ```
    
    ```cpp
    MyBox<int> intBox; //⭐ int 타입에 맞게 클래스가 정의됨
    intBox.set(10);
    std::cout << intBox.get();  // 출력: 10
    
    MyBox<std::string> strBox; //⭐ string 타입에 맞게 클래스가 정의됨
    strBox.set("hello");
    std::cout << strBox.get();  // 출력: hello
    ```
    
    즉, **타입마다 새로운 클래스가 컴파일 시점**에 만들어짐
    
</aside>

### ⚡이걸 특수화한다면?

```cpp
namespace std {
    template<typename T>
    // swap<Widget<T>> 요게 특수화임
    void swap<Widget<T>>(Widget<T>& a, Widget<T>& b) { 
        a.swap(b);  // ❗❗ 오류 발생 가능
    }
}
```

→ 보기에는 그럴싸해보이지만, **C++표준에서 허용되지 않는 문법**

**🔎  왜 안될까?**

<aside>

🚨 부분 특수화는 함수 템플릿에 허용되지 않음!

</aside>

- `swap<Widget<T>>` → 부분 특수화(partial specialization)임
- C++은 클래스 템플릿에 대해서는 부분 특수화가 ㄱㅊ음
- 하지만 함수 템플릿에는 부분 특수화 금지 → `std`에 새로운 템플릿을 추가하는 행위라서 그런대요

🧠 **그럼 어떻게 할까?**

**→ 오버로딩을 사용하자~**

```cpp
namespace std {
    template<typename T>
    void swap(Widget<T>& a, Widget<T>& b) {
        a.swap(b);
    }
}
```

- 이건 특수화가 아님
- `template<>` 없이 그냥 `template<typename T>`를 선언함
- 이건 오버로딩(overloading)이지 특수화(specialization)이 아님
- 이건 **허용됨**

### ⚡ 안전한 권장 방법: ADL 기반 사용자 정의 swap

여기 항목에 나온 `Widget` 관련 기능을 전부 `WidgetStuff` 네임스페이스에 때려박아라

```cpp
namespace WidgetStuff {
    template<typename T>
    class Widget {
    public:
        void swap(Widget& other) { // swap 멤버 함수
            std::swap(pImpl, other.pImpl);
        }
    };

    template<typename T> // 비멤버 swap 함수
	    void swap(Widget<T>& a, Widget<T>& b) { // std 네임스페이스의 일부가 아님
        a.swap(b);
    }
}
```

- 자신의 네임스페이스에 `swap` 정의
- 이러면 ADL(Augument-Dependent Lookup)이 `WidgetStuff::swap`을 자동으로 찾아줌

ADL(Augument-Dependent Lookup): 인자 기반 탐색?

- 어떤 함수에 어떤 타입의 인자가 있으면, 그 함수의 이름을 찾기 위해 해당 타입의 인자가 위치한 네임스페이스 내부의 이름을 탐색해 들어간다는 간단한 규칙 → 스코프

## ⚡ 만약 고객이 사용한다면?

고객이 사용할 수 있는 swap의 종류는 3가지

- **`std::swap`의 일반 버전**
- **`std::swap`의 특수화 버전 (T에 특화된)**
- **`T` 타입에 맞춰 따로 정의된 사용자 정의 `swap`**

### 😵 그냥 `swap(obj1, obj2);` 하면?

- C++의 **이름 탐색 규칙(ADL)**에 따라,
- 컴파일러는 다음 순서로 찾음:
    1. **obj1과 obj2 타입(T)의 네임스페이스에 정의된 `swap`이 있는지 확인**
    2. 없으면 `std::swap`을 찾으려고 함 (근데 `std::swap`은 `std` 안에 있으니까 안 보일 수도 있음)

### ✅ 그래서 이렇게 써야함

```cpp
template<typename T>
void doSomething(T& obj1, T& obj2) {
    using std::swap;  // ⭐ 이 줄이 핵심!
    swap(obj1, obj2); // 이렇게 하면 T 전용 → 없으면 std::swap
}
```

### 이 코드의 동작 방식:

- `using std::swap;` 선언으로 `std::swap`을 **현재 스코프로 끌어옴**
- 그다음 `swap(obj1, obj2);`를 호출하면
    - 먼저 T 타입에 특화된 swap이 있는지 확인
    - 없으면 `std::swap`을 호출

👉 즉, **T 전용 버전이 있으면 그것을 쓰고**, 없으면 **std::swap**을 쓰게 됨

### 🚨 잘못된 코드 예시

```cpp
std::swap(obj1, obj2);  // ❌ 무조건 std::swap만 호출됨
```

이렇게 하면 **항상 std::swap만 호출**됨

→ T 타입 전용 swap이 있어도 절대 호출되지 않음

→ **확장성/성능 저하/예외 안전성 문제 발생할 수 있음**

## 🔥 swap이 예외를 던지지 않도록 만들어야 하는 이유?

- swap은 다양한 상황에서 **예외 안전성 보장**을 위해 자주 사용됨
- 특히 **강한 예외 보장(strong exception guarantee)** 을 지키기 위해 swap이 중요함
- 만약 `swap`이 예외를 던지면, 예외 안전성이 깨질 수 있음 → 프로그램 불안정
- 그래서 swap은 반드시 예외를 던지지 않는 방식으로 구현해야 함
    
    (대부분의 swap은 **포인터 교환만 하기 때문에** 예외 안 생김)