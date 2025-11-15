# 항목39: private 상속은 심사숙고해서 구사하자 - 작성자: 신동욱

### 1. public 상속 vs private 상속

| 구분 | public 상속 | private 상속 |
| --- | --- | --- |
| 의미 | **is-a** 관계 (A는 B의 일종이다) | **is-implemented-in-terms-of** 관계 (A는 B를 이용해 구현된다) |
| 변환 | 파생 클래스 → 기본 클래스 **암시적 변환 가능 (업캐스팅 가능)** | 파생 클래스 → 기본 클래스 **암시적 변환 불가능** |
| 멤버 접근 | 기본 클래스의 `public`/`protected` 멤버 유지 | 기본 클래스의 모든 멤버가 **private으로 바뀜** |
| 사용 목적 | 인터페이스 확장, 다형성 구현 | **구현 재사용** (interface 상속 아님) |

---

### 2. private 상속의 의미

- is-implemented-in-terms-of
    
    → “B의 기능을 이용해 D를 구현한다”는 뜻.
    
    → 즉, **상속은 구현 편의를 위한 수단**이며, “개념적 관계”를 의미하지 않는다.
    
    → 기본 클래스는 단순히 **구현 세부사항**일 뿐이다.
    
- 따라서, **설계** 단계에서는 거의 의미가 없고, **구현** 단계에서만 유용하다

---

### 3. private 상속의 동작 규칙

1. 파생 클래스 객체는 기본 클래스 객체로 암시적 변환되지 않는다.
    
    → `Student`가 `Person`을 private 상속하면 `eat(s)` 는 에러.
    

```cpp
class Person {...};

class Student: private Person {...}; //private 상속

void eat(const Person& p); //사람은 먹는다

void study(const Student& s); //공부는 학생만 할 수 있다

Person p; //Person 인스턴스 p
Student s; //Student 인스턴스 s

eat(p); //가능

eat(s); //에러!
```

1. 기본 클래스의 모든 멤버는 **파생 클래스 안에서 private이 된다.**

---

### 4. private 상속을 사용할 때

- 객체 합성(composition)으로도 구현 가능하지만, 아래의 경우에는 private 상속이 필요할 수 있다
    1. 기본 클래스의 protected 멤버에 접근해야 할 때
    2. 기본 클래스의 가상 함수를 재정의해야 할 때
    3. 공간 최적화가 필요한 아주 특수한 경우

---

### 5. 예시: Widget과 Timer

- 자 Widget 객체를 사용하여 응용프로그램을 하나 만들고 있다고 가정하자.
- 근데 Widget 멤버 함수의 호출 횟수같은 것들도 알고 싶고 실행 시간이 지남에 따라 호출 비용이 어떻게 변하는지 알고 싶다 ⇒ 즉, 클래스의 프로파일(profile)을 알고싶다
- 각 멤버 함수가 몇 번이나 호출되는지를 추적하기 위해 Widget 클래스를 직접 손보기로 하자
- 멤버 함수의 호출 횟수 정보는 프로그램이 실행되는 도중에 주기적으로 점검하도록 만들텐데, 필요하다면 이 정보 외에 각 Widget 객체의 값과 더불어 우리 생각에 유용하다고 여겨지는 다른 자료들도 넣을 수 있을 것
- 이 작업을 위해 일종의 타이머를 하나 설치해 놓는다
- 함수 사용 통계를 수집할 때를 알려 주는 용도로 이 타이머를 쓰자는거임
- 근데 Timer를 만들어 놓은게 있다

```cpp
class Timer {
public:
	explicit Timer(int tickFrequency);
	
	virtual void onTick() const; // 일정 시간이 경과할 때마다 자동으로
															// 이것이 호출됨
	...
	};
```

- Timer 객체는 반복적으로 시간을 경과시킬 주기를 우리가 정해줄 수 있다
- 일정 시간이 경과할 때마다 가상함수를 호출하도록 되어 있다
- 이 가상 함수를 재정의해서, Widget 객체의 현재 상태를 점검하면 되겠네!
- 아 그럼 Widget 클래스에서 Timer의 가상 함수를 재정의할 수 있어야 하므로, Widget 클래스는 어쨌든 Timer에서 상속을 받아야 함
- 근데 지금 상황에서 public 상속은 맞지 않음. Widget이 Timer의 일종(is-a)가 아니기 때문
- …

```cpp
class Timer{
public:
	virtual void onTick() const;
};

class Widget : private Timer {
private:
	virtual void onTick() const; // Widget 사용 자료 등을 수집
	...
};
```

왜 private 상속인가?

- Widget은 Timer의 **일종(is-a)** 이 아니다.
- `onTick()` 은 **Widget 사용자가 직접 호출하면 안되는 함수**이다.
- 따라서 `Timer`의 인터페이스를 노출하지 않기 위해 **private 상속**이 적절하다.

자자자 근데 이거 객체 합성으로 해도 되거든여

- Timer로부터 public 상속을 받은 클래스를 Widget안에 private 중첩 클래스로 선언해 놓고, 이 클래스에서 onTick을 재정의한 다음, 이 타입의 객체 하나를 Widget 안에 데이터 멤버로서 넣는 것

```cpp
class Widget {
private:

	class WidgetTimer : public Timer {
	public:
		virtual void onTick() const;
		...
	};
	
	WidgetTimer timer;
	...
};
```

- private 상속만 써서 만든 설계와 비교하면 상당히 복잡
- ‘하나의 설계 문제에 대한 접근 방법이 꼭 하나만 있는 것은 아니다’
- ‘여러 가지 방법을 실제로 고민하는 습관을 들이는 것이 좋다’
- private 상속 대신에 public 상속에 객체 합성 조합이 더 좋은 이유?
    1. Widget 클래스를 설계하는 데 있어서 파생은 가능하게 하되, 파생 클래스에서 onTick을 재정의할 수 없도록 설계 차원에서 막고 싶을 때 유용
        
        Widget을 Timer로부터 상속시킨 구조라면 이게 안됨
        
        private 상속해도 안됨
        
    2. Widget의 컴파일 의존성을 최소화하고 싶을 때 좋다
        
        Widget이 Timer에서 파생된 상태라면, Widget이 컴파일될 때 Timer의 정의도 끌어올 수 있어야 하기 때문에, Widget의 정의부 파일에서 Timer.h 같은 헤더를 #include 해야 할지도 모름
        
        반면, 지금의 설계에서는 WidgetTimer의 정의를 Widget으로부터 빼내고 Widget이 WidgetTimer 객체에 대한 포인터만 갖도록 만들어 두면, WidgetTimer 클래스를 간단히 선언하는 것만으로도 컴파일 의존성을 슬쩍 피할 수 있다
        
        TTimer에 관련된 어떤 것도 #include 할 필요가 없으니까용
        
        규모가 큰 시스템을 만들 때 이런 구성요소 분리는 굉장히 중요해질 수 있는 요소랍니다
        

---

### 정리

- private 상속을 사용하는 때
    - 기본 클래스의 비공개 부분에 파생 클래스가 접근해야 한다거나
    - 가상 함수를 한 개 이상 재정의해야 할 경우
- 이때 is-a 관계가 아니라 is-implemented-in-terms-of 관계다
- 이제 객체 합성보다 private 상속을 써야 하는 경우를 아라보자

---

### private 상속이 유용한 경우 — 공간 최적화

☑️ **문제:**

C++은 **크기가 0인 객체를 허용하지 않음.**

그래서 데이터 멤버가 없는 “공백 클래스(empty class)”라도 **1바이트 이상** 차지함.

```cpp
class Empty {};
class HoldsAnInt {
private:
	int x;
	Empty e;
};
```

⇒ `sizeof(HoldsAnInt) > sizeof(int)`

이상하게도 `Empty`가 1바이트를 차지해서 전체 크기가 커짐.

☑️ 해결:

“공백 클래스”를 **데이터 멤버로 넣지 말고 상속**시키면 된다.

```cpp
class HoldsAnInt : private Empty {
private:
	int x;
};
```

⇒ `sizeof(HoldsAnInt) == sizeof(int)`

이 최적화를 **EBO(Empty Base Optimization)** 이라 부름.

대부분의 컴파일러가 지원하며, **단일 상속(single inheritance)** 에서만 적용됨.

☑️ EBO는 예외적인 상황이다.

- “진짜로 텅 빈 클래스”는 거의 없음.
- `typedef`, `enum` , 정적 멤버, 비가상 함수 등은 종종 존재함.
- STL의 `unary_function`, `binary_function` 등이 이런 식의 **공백 클래스**를 활용함.
- **EBO 덕분에 상속해도 메모리 낭비가 거의 없음.**

☑️ 그럼에도 불구하고…

- 대부분의 경우 is-a 관계는 `public` 상속으로 표현해야 함.
- `private` 상속은 설계 복잡도를 높이고, 관계를 모호하게 만듦.
- **가능하면 객체 합성(composition)**으로 표현하는 것이 명확하고 유지보수에 유리함.

### 결론 : “private 상속을 심사숙고해서 구사하자”