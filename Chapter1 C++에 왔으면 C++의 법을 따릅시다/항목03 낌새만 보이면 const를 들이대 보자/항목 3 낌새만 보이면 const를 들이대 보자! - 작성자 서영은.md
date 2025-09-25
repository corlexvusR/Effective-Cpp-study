# 항목 3: 낌새만 보이면 const를 들이대 보자! - 작성자: 서영은

<aside>

# 💡이것만은 잊지말자!

- const를 붙여 선언하면 컴파일러가 사용상의 에러를 잡는 데 도움을 준다.
- 컴파일러 쪽에서 보면 비트수준 상수성을 지켜야 하지만, 개념적인(논리적인) 상수성을 사용해서 프로그래밍해야 한다.
- 상수 멤버 및 비상수 멤버 함수가 기능적으로 서로 똑같게 구현되어 있을 경우에는 코드 중복을 피하기 위해 비상수 버전이 상수 버전을 호출하도록 만들자.
</aside>

---

# 📌 const를 쓸 수 있는 경우 (클래스 내외)

클래스 바깥

- 전역 혹은 네임스페이스 유효범위의 상수를 선언(정의)하는 데 쓸 수 있음.
- 파일, 함수, 블록 유효범위에서 static으로 선언한 객체에도 const를 붙일 수 있음

클래스 내부

- 정적 멤버 및 비정적 데이터 멤버 모두를 상수로 선언할 수 있음
- 포인터 자체를 상수로, 포인터가 가리키는 데이터를 상수로 지정할 수 있음.

# 📌 const와 포인터

```cpp
char greeting[] = "Hello";

char *p = greeting;  // 비상수 포인터, 비상수 데이터
const char *p = greeting;  // 비상수 포인터, 상수 데이터
char * const p = greeting; // 상수 포인터, 비상수 데이터
const char * const p = greeting; // 상수 포인터, 상수 데이터
```

const가 *표의 **`왼쪽`**에 있음 ⇒ **포인터가 가리키는 대상이 상수**

const가 *표의 **`오른쪽`**에 있음 ⇒ **포인터 자체가 상수**

const가 *표의 **`양쪽`**에 다 있음 ⇒ **포인터가 가리키는 대상 및 포인터가 다 상수**

포인터가 가리키는 대상을 상수로 만들때 const 사용하는 스타일은 조금씩 다름.

아래처럼 다르게 써도 의미는 같음.

```cpp
// 두 함수 모두 상수 Widget 객체에 대한 포인터를 매개변수로 취한다.
void f1(const Widget *pw);
void f2(Widget const *pw);
```

# 📌 const와 STL 반복자(iterator)

```cpp
std::vector<int> vec;

const std::vector<int>::iterator iter = vec.begin();
							// iter는 T* const처럼 동작 (포인터 자체가 상수)
*iter = 10;  // iter가 가리키는 대상 변경: 가능
++iter;      // 불가능. iter는 상수

std::vector<int>::const_iterator cIter = vec.begin();
							// cIter는 const T*처럼 동작 (포인터가 가리키는 대상이 상수)
*cIter = 10;  // 불가능. *cIter는 상수
++cIter;      // cIter 변경: 가능
```

STL 반복자(iterator)는 포인터를 본뜬 것이라 기본적인 동작 원리가 T* 포인터와 매우 흡사.

어떤 반복자를 const로 선언하는 것은 포인터를 상수로 선언하는 것과 같다.

반복자는 자신이 가리키는 대상이 아닌 것을 가리키는 경우가 불가능하지만, 반복자가 가리키는 대상 자체를 변경하는 것은 가능.

만약 변경이 불가능한 객체를 가리키는 반복자, 즉 const T* 포인터의 STL 대응물이 필요하다면 const_iterator를 쓰면 됨.

# 📌 const와 함수

함수 선언문에 있어서 const는 함수 반환 값, 각각의 매개변수, 멤버 함수 앞에 붙을 수 있고, 함수 전체에 대해 const의 성질을 붙일 수 있음.

함수 반환 값을 상수로 정하면, 안전성이나 효율을 포기하지 않고도 사용자측의 에러 돌발 상황을 줄이는 효과.

```cpp
class Rational {...};
const Rational operator*(const Rational& lhs, const Rational& rhs);
```

# 📌 상수 멤버 함수

멤버 함수에 붙는 const의 역할 ⇒ 해당 멤버 함수가 상수 객체에 대해 호출될 함수라는 사실을 알려주는 것

## 상수 멤버 함수가 중요한 이유

- **클래스의 인터페이스를 이해하기 좋게 하기 위해**
    - 클래스로 만들어진 객체를 변경할 수 있는 함수는 무엇이고, 변경할 수 없는 함수는 무엇인가를 사용자 쪽에서 알고 있어야 하는 것
- **이 키워드를 통해 상수 객체를 사용할 수 있게 하자는 것**
    - 코드의 효율을 위해 아주 중요한 부분.
    - C++ 프로그램의 실행 성능을 높이는 핵심 기법 중 하나가 객체 전달을 상수 객체에 대한 참조자(reference-to-const)로 진행하는 것이기 때문.
    - 다만 이 기법이 가능하려면 상수 상태로 전달된 객체를 조작할 수 있는 const 멤버 함수, 즉 상수 멤버 함수가 준비되어 있어야 함.

**`const 키워드가 있고 없고의 차이만 있는 멤버 함수들은 오버로딩이 가능함`** 

```cpp
class TextBlock { 
public:
	...
	const char& operator[](std::size_t position) const // 상수 객체에 대한 operator[]
	{ return text[position]; }
	
	char& operator[](std::size_t position)  // 비상수 객체에 대한 operator[]
	{ return text[position]; }
	
private: 
	std::string text;
};

TextBlock tb("Hello"); // TextBlock::operator[]의 비상수 멤버를 호출
std::cout << tb[0];

const TextBlock ctb("World"); // TextBlock::operator[]의 상수 멤버를 호출합니다.
std::cout << ctb[0];
```

**실제 프로그램에서 상수 객체가 생기는 경우**

- 상수 객체에 대한 포인터로 객체가 전달될 때
- 상수 객체에 대한 참조자로 객체가 전달될 때

```cpp
void print(const TextBlock& ctb)  // 이 함수에서 ctb는 상수 객체로 쓰임
{
	std::cout << ctb[0];  // TextBlock::operator[]의 상수 멤버를 호출
	...
}
```

operator[]를 `오버로드(overload)`해서 각 버전마다 반환 타입을 다르게 가져갔기 때문에 TextBlock의 상수 객체와 비상수 객체의 쓰임새가 달라짐

`오버로드란`? 같은 이름의 함수에 매개변수를 다르게 사용하여 매개 변수마다 다른 함수가 실행되는 것.

- 오버로딩(overloading)와 오버라이딩(overriding)
    
    **오버로드(overloading):** 같은 이름의 함수에 매개변수를 다르게 사용하여 매개 변수마다 다른 함수가 실행되는 것.
    
    **오버라이드(overriding):** 상속 관계에서 부모클래스의 함수를 사용하지 않고 다른 기능을 실행할 때 자식클래스에 같은 이름, 매개변수로 재정의하여 사용하는 것
    
    **출처** https://hwan1402.tistory.com/87
    

```cpp
std::cout << tb[0];  // 비상수 버전의 TextBlock 객체를 읽음
tb[0] = 'x';         // 비상수 버전의 TextBlock 객체를 씀
std::cout << ctb[0]; // 상수 버전의 TextBlock 객체를 읽음
ctb[0] = 'x';        // 컴파일 에러! 상수 버전의 TextBlock 객체에 쓰기는 불가능
```

4번째 줄에서 에러가 발생한 이유

⇒ 반환 타입(return type) 때문. operator[] 호출이 잘못된 것은 아님.  
const char& 타입에 대한 대입 연산을 시도해서.

⚠️ **주의할 점**

operator[]의 비상수 멤버는 char의 참조자(reference)를 반환함. char 하나만 쓰면 안됨.

만일 그냥 char를 반환하게 만들었다면 2번째 줄이 컴파일되지 않음.

왜? ⇒ 기본제공 타입을 반환하는 함수의 반환 값을 수정하는 일은 절대로 있을 수 없기 때문.

만일 통한다고 해도 ‘값에 의한 반환’을 수행하는 C++의 성질 때문에 tb.text[0]의 사본이 수정됨.

# 📌 비트수준 상수성(bitwise constness)과 논리적 상수성(logical constness)

## 비트수준 상수성(bitwise constness)

- 다른말로 물리적 상수성(physical constness)
- 어떤 멤버 함수가 그 객체의 어떤 데이터 멤버도 건드리지 않아야(정적 멤버는 제외) 그 멤버 함수가 const임을 인정하는 개념.
- 즉, 그 객체를 구성하는 비트들 중 어떤 것도 바꾸면 안된다는 것.

비트수준 상수성을 사용하면 상수성 위반을 발견하는 데 힘이 많이 들지 않음

왜? ⇒ 컴파일러는 데이터 멤버에 대해 대입 연산이 수행되었는지만 보면 되기 때문

그러나 제대로 const로 동작하지 않는데도 비트수준 상수성 검사를 통과하는 경우가 있음

어떤 포인터가 가리키는 대상을 수정하는 멤버 함수들 중 상당수가 그러함.

하지만 그 포인터가 객체의 멤버로 들어있으면 이 함수는 비트수준 상수성을 갖는 것으로 판별됨.

이것 때문에 문제가 생길 수 있음

```cpp
class CTextBlock {
public:
	...
	char& operator[](std::size_t position) const // 부적절한 (그러나 비트수준 상수성이
	{ return pText[position]; }                  // 있어서 허용되는 operator[]의 선언)
	
private:
	char *pText;
};
```

operator[] 함수가 상수 멤버 함수로 선언되어 있음. 해당 객체의 내부 데이터에 대한 참조자를 반환함.

pText는 건드리지 않아서 괜찮아 보임. 그러나 아래와 같은 경우가 생김.

```cpp
const CTextBlock cctb("Hello"); // 상수 객체 선언
char *pc = &cctb[0];     // 상수 버전의 operator[] 호출하여 내부 데이터에 대한 포인터 얻음
*pc = 'J';               // cctbs는 이제 "Jello" 값이 됨
```

값이 변해버림.

## 논리적 상수성(logical constness)

- 위와 같은 상황을 보완하는 대체 개념.
- 상수 멤버 함수라고 해서 객체의 한 비트도 수정할 수 없는 것이 아니라 일부 몇 비트 정도는 바꿀 수 있되, 그것을 사용자측에서 알아채지 못하게 하면 상수 멤버 자격이 있다는 것

**예시**

```cpp
class CTextBlock {
public:
	...
	std::size_t length() const;
	
private:
	char *pText;
	std::size_t textlength;   // 바로 직전에 계산한 텍스트 길이
	bool lengthIsValid;       // 이 길이가 현재 유효한가?
};

std::size_t CTextBlock::length() const
{
	if(!lengthIsValid) {
		textlength = std::strlen(pText);  // 에러! 상수 멤버 함수 안에서는 
		lengthIsValid = true;             // textLength 및 lengthIsValid에 대입할 수 없음
	}
	
	return textlength;
}
```

length() 함수가 비트수준 상수성과 떨어져 있음

왜?? ⇒ textLength 및 lengthIsValid가 바뀔 수 있으니까

하지만 CTextBlock의 상수 객체에 대해서는 아무 문제가 없어야 할 것 같은 코드임.

그러나 컴파일러는 에러남. 컴파일러 에러를 통과하려면 비트 수준의 상수성이 지켜져야 함.

### ✅ 해결방법. mutable 키워드 사용

```cpp
class CTextBlock {
public:
	...
	std::size_t length() const;
	
private:
	char *pText;
	mutable std::size_t textlength;   // 이 멤버들은 어떤 순간에도 수정이 가능
	mutable bool lengthIsValid;       // 상수 멤버 함수 안에서도 수정 가능
};

std::size_t CTextBlock::length() const
{
	if(!lengthIsValid) {
		textlength = std::strlen(pText);  // 이제 에러 안남
		lengthIsValid = true;             // 에러 없음
	}
	
	return textlength;
}
```

**참고** https://modoocode.com/253

# 📌 상수 멤버 및 비상수 멤버 함수에서 코드 중복 현상

```cpp
class TextBlock { 
public:
	...
	const char& operator[](std::size_t position) const
	{
		...  // 경계 검사
		...  // 접근 데이터 로깅
		...  // 자료 무결성 검증
		return text[position];
	}
	
	char& operator[](std:：size_t position)
	{
		...  // 경계 검사
		...  // 접근 데이터 로깅
		...  // 자료 무결성 검증
		return text[position];
	}
	
 private:
	 std::string text;
 };
```

## ✅ 해결방법. 비상수 버전이 상수 버전을 호출하게 구현하자

```cpp
class TextBlock { 
public:
	...
	const char& operator[](std::size_t position) const  // 이전과 동일
	{
		...  // 경계 검사
		...  // 접근 데이터 로깅
		...  // 자료 무결성 검증
		return text[position];
	}
	
	char& operator[](std:：size_t position)  // 상수 버전 op[]를 호출하고 끝
	{
		return const_cast<char&>(                // op[]의 반환 타입에 캐스팅 적용 (const 없앰)
						static_cast<const TextBlock&>    // *this의 타입(TextBlock&)에 const를 붙임
							(*this)[position]              // op[]의 상수 버전을 호출
					);
	}
	...
 };
```

TextBlock&에 const를 붙여서 op[]의 상수 버전을 호출

→ op[]의 상수 버전의 반환 타입에 캐스팅을 적용해서 const를 없앰