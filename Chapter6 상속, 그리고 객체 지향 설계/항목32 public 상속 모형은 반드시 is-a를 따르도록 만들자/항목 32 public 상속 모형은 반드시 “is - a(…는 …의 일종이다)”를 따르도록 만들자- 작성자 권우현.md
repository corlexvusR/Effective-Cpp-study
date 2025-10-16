# 항목 32: public 상속 모형은 반드시 “is - a(…는 …의 일종이다)”를 따르도록 만들자 - 작성자: 권우현

<aside>

# 💡이것만은 잊지말자!

- public 상속의 의미는 “is-a(…는 …의 일종)” 입니다. 기본 클래스에 적용되는 모든 것들이 파생 클래스에 그대로 적용되어야 합니다. 왜냐하면 모든 파생 클래스 객체는 기본 클래스 객체의 일종이기 때문입니다.
</aside>

---

# 1. 개요

c++로 객체 지향 프로그래밍을 하면서 다른 건 잊더라고 꼭 잊지 말아달라…!

public 상속은 “ is-a(…는 …의 일종이다)”를 의미한다는 이야기

if )  class D (”Derived”) 를 class B (”Base”) 로부터 public 상속을 통해 파생시켰다면 , 

c++ 컴파일러에게 이렇게 말한 것과 같습니다.

⇒ D 타입으로 만들어진 모든 객체는 또한 B 타입의 객체이지만 **그 반대는 되지 않는다**.

B : 일반적인 개념, D : 특수한 개념

따라서 B 타입의 객체가 쓰일 수 있는 곳에는 D 타입의 객체도 쓰일 수 있음. (D 타입의 모든 객체는 B 타입의 객체도 되기 때문에)

하지만 반대인, D 타입이 필요한 부분에 B 타입의 객체를 쓰는 것은 불가능 함. 

⇒ 모든 D 는 B의 일종이지만 (D is a B), B는 D 의 일종이 아니기 때문입니다.

# 2. public 상속

```jsx
class Person { ... };
class Student : public Person { ... };
```

Person 타입 (Person에 대한 포인터, 참조자)의 인자를 기대하는 함수는 Student 객체(student 에 대한 포인더, 참조자) 도 받아들 일 수 있다.

```cpp
void eat(const Person& p);
void study(const Student& s);

Person p;
Student s;
eat(p); // 가능
eat(s);  //가능
study(s);  //가능
study(p);  //불가능

=> public 상속에서만 통하는 이야기. 
=> Student 와 Person 이 public 상속 관계를 가져야 함.

private 상속은 의미가 다르고, protected 상속은 의미가 아리아리하대;;
```

# 3. 직관에 속지마라!

## 1) 펭귄

직관 때문에 판단을 잘못하는 경우

펭귄이 새의 일종

새라는 개념만 보면 새가 날 수 있다는 것도 사실

이것을 c++로 표현해 본다면,

```cpp
class Bird {
public:
	virtual void fly(); // 새는 날 수 있습니다.
	...
};

class Penguin: public Bird { // 펭귄은 새입니다.
...
};

=> 펭귄은 날 수 있다는 모순에 빠짐. 
=> 사람의 자연어에 낚인 것. 
더 명확히 해봅시다.
```

```cpp
class Bird {
	...  //fly 함수가 선언되지 않았습니다.
};

class FlyingBird: public Bird{
public:
	virtual void fly();
	...
};

class Penguin: public Bird{
	...  //fly 함수가 선언되지 않았습니다.
}
```

### 1) 안 좋은 대처의 예

```cpp
void error(const std::strings$ msg); //어딘가에 정의되어 있을 것

class Penguin: public Bird{
public: 
	virtual void fly() {error("Attempt to make a penguin fly!");}
	...
}
```

⇒ “펭귄을 날 수 없다” 가 아닌 “펭귄은 날 수 있다. 그러나 펭귄이 실제로 날려고 하면 에러가 난다”

⇒ fly 함수의 재 정의로 런타임 에러를 내는 방법. 

### 2) 좋은 대체의 예

- 펭귄은 날 수 없다 라는 제약사항이 들어간 것.
- 비행과 관련된 함수를 정의하지 않으면 됨.

```cpp
class Bird {
	...  // fly 함수가 선언되지 않음.
};

class Penguin: public Bird {
	...  // fly 함수가 선언되지 않음.
};
```

```cpp
Penguin p;
p.fly();  // 에러 발생!
```

⇒ 위의 안좋은 예랑은 다른 모습 ⇒ 컴파일러가 [p.fl](http://p.fly)y 를 호출하는 것에 대해 에러가 나지 않는것

⇒ 밑의 예시는 유효하지 않은 코드를 컴파일 단계에서 막아주는 인터페이스 (좋은 인터페이스!)

## 2) 직사각형과 정사각형.

### (1) 클래스를 생성할 때 생각해야 할 점

직관으로 생각해서는 정사각형은 직사각형의 부분집합이기 때문에, 

Square(정사각형) 클래스는 Rectangle(직사각형) 클래스로 부터 상속을 받아야 할 것 처럼 보인다.

 

<aside>
💡

하지만 기억하자!

public 상속은 기본 클래스 객체가 가진 **모든 것들이** 파생 클래스 객체에도 그대로 적용된다고 단정하는 상속이다. 

직사각형의 성질 중 어떤 것 (가로 길이가 세로 길이에 상관없이 바뀔 수 있습니다)은 정사각형 (가로와 세로 길이가 같아야 합니다)에 적용할 수 없다는 점.

⇒ 그러므로 이러한 단정은 참이 될 수 가 없어서 , 이 둘의 관계를 public 상속을 써서 표현하려고 하면 틀리는 것이 당연함.

</aside>

### (2) 잘못된 예시

정사각형이 직사각형에서 상속이 된다고 생각하고 짠 코드를 보면, 컴파일러 수준에서는 문법적 하자가 없기 때문에 코드가 통과되게 됨. 

```cpp
class Rectangle {
public:
	virtual void setHeight (int newHeight);
	virtual void setWidth (int newWidth);
	
	virtual int height() const; // 현재의 값을 반환합니다.
	virtual int width() const;
	...
};
void makeBigger(Rectanglr& r)
{
	int oldHeight = r.height();
	r.setWidth(r.width() + 10); // r의 가로 길이에 10을 더합니다.
	assert(r.height() == oldHeight); // r의 세로길이가 변하지 않는다는 조건에 단정문을 걸어둡니다.
}
```

⇒ 단정문은 실패할 일이 없음 (assert)

- assert 에 관한 간단 설명
    
    
    assert(조건식)형태로 쓰이고, 조건식이 ,**거짓(false)** 이면 프로그램을 **즉시 중단** 시키고 에러를 띄움.
    
    “가로 길이를 늘려도 세로 길이는 변하지 않아야 한다”라는 **불변식(invariant)**을 확인하는 것임.
    

⇒ makeBigger 함수는 r의 가로길이만 변경할 뿐이고, 세로 길이는 바뀌지 않는다.

```cpp
class Square: public Rectangle{...};

Square s;
...
assert(s.width() == s.height()); // 이 단정문은 모든 정사각형에 대해 참이어야 함

makeBigger(s); // 상속된 것이므로, s는 Rectangle의 일종, s의 넓이를 늘릴 수 있음

assert(s.width() == s.height()); // 이번에도 이 단정문이 모든 정사각형에 대해 참이어야 한다.

```

---

뭔가 이상하다는 것을 생각해보아야 함. 

- 1)makeBigger 함수를 호출하기 전에, s 의 세로 길이는 가로 길이와 같아야 한다.
- 2)makeBigger 함수가 실행되는 중에, s의 가로 길이는 변하는데 세로 길이는 변하지 않아야 한다.
- 3)makeBigger 함수에서 복귀한 후에, s 의 세로 길이는 역시 가로 길이와 같아야 한다. (s 가 makeBigger로 넘겨질 때 참조에 의한 전달이 되고 있음. makeBigger함수가 변경하는 것은 s 그 자체)

# 4. 클래스들 사이에 맺을 수 있는 관계

- is-a (이번 항목)
- has-a
- is-implemented-in-terms-of

⇒ 항목 38, 39 에 있음!